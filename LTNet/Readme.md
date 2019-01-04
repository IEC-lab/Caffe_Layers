# LTNet

参考文献[Facial expression recognition with inconsistent datasets](https://arxiv.org/abs/1712.07195)

## Transplant

### Transplant the ./code files to you repo.

- include/caffe/layers/multilabel_image_data_layer.hpp
- include/caffe/layers/inner_product_as_prob_layer.hpp
- src/caffe/layers/multilabel_image_data_layer.cpp
- src/caffe/layers/inner_product_as_prob_layer.cpp
- src/caffe/layers/inner_product_as_prob_layer.cu

### modify caffe.proto

*可参考本目录下的./code/add_to_caffe.proto*

(1) 将如下代码加在 **message LayerParameter**里面

```
optional MultilabelImageDataParameter multilabel_image_data_param = 155;
optional InnerProductAsProbParameter inner_product_as_prob_param = 156;
```

（2）在caffe.proto中添加如下代码

```
message MultilabelImageDataParameter {
  // Specify the data source.
  optional string source = 1;
  // Specify the batch size.
  optional uint32 batch_size = 4 [default = 1];
  // The rand_skip variable is for the data layer to skip a few data points
  // to avoid all asynchronous sgd clients to start at the same point. The skip
  // point would be set as rand_skip * rand(0,1). Note that rand_skip should not
  // be larger than the number of keys in the database.
  optional uint32 rand_skip = 7 [default = 0];
  // Whether or not ImageLayer should shuffle the list of files at every epoch.
  optional bool shuffle = 8 [default = false];
  // It will also resize images if new_height or new_width are not zero.
  optional uint32 new_height = 9 [default = 0];
  optional uint32 new_width = 10 [default = 0];
  // Specify if the images are color or gray
  optional bool is_color = 11 [default = true];
  // DEPRECATED. See TransformationParameter. For data pre-processing, we can do
  // simple scaling and subtracting the data mean, if provided. Note that the
  // mean subtraction is always carried out before scaling.
  optional float scale = 2 [default = 1];
  optional string mean_file = 3;
  // DEPRECATED. See TransformationParameter. Specify if we would like to randomly
  // crop an image.
  optional uint32 crop_size = 5 [default = 0];
  // DEPRECATED. See TransformationParameter. Specify if we want to randomly mirror
  // data.
  optional bool mirror = 6 [default = false];
  optional string root_folder = 12 [default = ""];
  // assigne the number of labels
  optional uint32 label_num = 13 [default = 1];
}


message InnerProductAsProbParameter {
  optional uint32 num_output = 1; // The number of outputs for the layer
  optional FillerParameter weight_filler = 2; // The filler for the weight
  optional int32 axis = 3 [default = 1];
  optional float minvalue = 4[default = 0];
}
```



## Usage

*具体例子见./demo目录*

### train

训练数据是**多标签,**在最后的全连接层后面需要接一个softmax层作为hidden_gt，然后对每一个标签和hidden_gt作为InnerProductAsProb的操作，最后接softmaxwithloss的损失函数，以三标签为例

```
##======Hidden GT======

layer {
  name: "fc_19"
  type: "InnerProduct"
  bottom: "pool_19"
  top: "fc_19"
  param {
    lr_mult: 1.0
    decay_mult: 2.0
  }
  param {
    lr_mult: 1.0
    decay_mult: 0.0
  }
  inner_product_param {
    num_output: 10
    weight_filler {
      type: "xavier"
    }
    bias_filler {
      type: "constant"
      value: 0
    }
  }
}

layer {
  name: "hidden_gt"
  type: "Softmax"
  bottom: "fc_19"
  top: "hidden_gt"
}

##======split label to 3 labler====
layer {
  name: "splitlabel"
  type: "Slice"
  bottom: "label"
  top: "label1"
  top: "label2"
  top: "label3"
  slice_param {
    axis: 1
    slice_point: 1
    slice_point: 2
  }
}

##===== observe matrix ====
layer {
  name: "observe_fc_1"
  type: "InnerProductAsProb"
  bottom: "hidden_gt"
  top: "observe1"
  param{lr_mult: 1}
  inner_product_as_prob_param {
    num_output: 10
    weight_filler {
      type: "xavier"
    }
    minvalue: 0.01
  }
}

layer {
  name: "loss1"
  type: "SoftmaxWithLoss"
  bottom: "observe1"
  bottom: "label1"
  top: "loss1"
}

##===== observe matrix ====
layer {
  name: "observe_fc_2"
  type: "InnerProductAsProb"
  bottom: "hidden_gt"
  top: "observe2"
  param{lr_mult: 1}
  inner_product_as_prob_param {
    num_output: 10
    weight_filler {
      type: "xavier"
    }
    minvalue: 0.01
  }
}

layer {
  name: "loss2"
  type: "SoftmaxWithLoss"
  bottom: "observe2"
  bottom: "label2"
  top: "loss2"
}

##===== observe matrix ====
layer {
  name: "observe_fc_3"
  type: "InnerProductAsProb"
  bottom: "hidden_gt"
  top: "observe3"
  param{lr_mult: 1}
  inner_product_as_prob_param {
    num_output: 10
    weight_filler {
      type: "xavier"
    }
    minvalue: 0.01
  }
}

layer {
  name: "loss3"
  type: "SoftmaxWithLoss"
  bottom: "observe3"
  bottom: "label3"
  top: "loss3"
}

```

### test

测试数据是单标签的，最后一个全连接层直接接softmax层得到每一类的概率输出即可

```
layer {
  name: "fc_19"
  type: "InnerProduct"
  bottom: "pool_19"
  top: "fc_19"
  param {
    lr_mult: 1.0
    decay_mult: 2.0
  }
  param {
    lr_mult: 1.0
    decay_mult: 0.0
  }
  inner_product_param {
    num_output: 10
    weight_filler {
      type: "xavier"
    }
    bias_filler {
      type: "constant"
      value: 0
    }
  }
}

layer {
  name: "accuracy"
  type: "Accuracy"
  bottom: "fc_19"
  bottom: "label"
  top: "accuracy"
}
```

