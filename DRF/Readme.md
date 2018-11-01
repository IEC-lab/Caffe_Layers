# DeepRegressionForests

参考文献[Deep Regression Forests for Age Estimation](https://arxiv.org/abs/1712.07195)

## Transplant

### Transplant the ./code files to you repo.

(util)

- include/caffe/util/sampling.hpp
- src/caffe/util/sampling.cpp
- include/caffe/util/neural_decision_util_functions.hpp
- src/caffe/util/neural_decision_util_functions.cu

(training)

- include/caffe/layers/neural_decision_reg_forest_loss_layer.hpp
- src/caffe/layers/neural_decision_reg_forest_loss_layer.cpp
- src/caffe/layers/neural_decision_reg_forest_loss_layer.cu

(testing)

- include/caffe/layers/neural_decision_reg_forest_layer.hpp
- src/caffe/layers/neural_decision_reg_forest_layer.cpp
- src/caffe/layers/neural_decision_reg_forest_layer.cu

(Eigen)

- 将Eigen目录放在caffe/include目录下

### modify caffe.proto

*可参考本目录下的./code/caffe.proto*

(1) 将如下代码加在 **message LayerParameter**里面

```
optional NeuralDecisionForestParameter neural_decision_forest_param = 149;
optional LDLMetricParameter ldl_metric_param = 150;
optional CSParameter cs_param = 151;
```

（2）在caffe.proto中添加如下代码

```
message NeuralDecisionForestParameter {
	optional uint32 depth = 1 [default = 3];
	optional uint32 num_trees = 2 [default = 1];
	optional uint32 num_classes = 3 [default = 2];
	optional uint32 iter_times_class_label_distr = 4 [default = 20];
	optional uint32 iter_times_in_epoch = 7 [default = 20];
	optional string record_filename = 5 [default="Forest.Record"];
  optional uint32 axis = 6 [default = 1];
  optional bool debug_gpu = 8 [default = false];
  optional bool use_gpu = 9 [default = true];
  optional uint32 all_data_vec_length = 10 [default = 5];
  optional bool drop_out = 11 [default = false];
  optional string init_filename = 12 [default = "Leafnode.Init"];
  optional float scale = 13 [default = 100.0];
}

message LDLMetricParameter {
  enum LDLMetricType {
    KLD = 1;
    Clark = 2;
    Chebyshev = 3;
    Canberra = 4;
    Cosine = 5;
    Inter = 6;
    Fidelity = 7;
    Euclid = 8;
    Soren = 9;
    Square = 10;
  }
  optional LDLMetricType metric_type = 1 [default = KLD];
}

message CSParameter {
  optional int32 lll = 1 [default = 5];
}
```



## Usage

*具体例子见./demo目录*

### train

在最后的全连接层后面接上回归决策树loss layer

```
layer {
  name: "probloss1"
  type: "NeuralDecisionRegForestWithLoss"
  bottom: "fc8"
  bottom: "label"
  top: "loss"
  param {
    lr_mult: 0.0
    decay_mult: 0.0
  }
  param {
    lr_mult: 0.0
    decay_mult: 0.0
  }
  param {
    lr_mult: 0.0
    decay_mult: 0.0
  }
  neural_decision_forest_param {
    depth: 6
    num_trees: 5
    num_classes: 1
    iter_times_class_label_distr: 20
    record_filename: "F_morph.record"
    iter_times_in_epoch: 50
    all_data_vec_length: 50
    drop_out: false
    init_filename: "init"
  }
}
```

### test

将train.prototxt里面的损失层替换成如下

```
layer {
  name: "probloss1"
  type: "NeuralDecisionRegForest"
  bottom: "fc8"
  top: "pred"
  neural_decision_forest_param {
    depth: 6
    num_trees: 5
    num_classes: 1
  }
}
```

```
@inproceedings{shen2018DRFs,
  author = {Wei Shen and Yilu Guo and Yan Wang and Kai Zhao and Bo Wang and Alan Yuille},
  booktitle = {Proc. CVPR},
  title = {Deep Regression Forests for Age Estimation},
  year = {2018}
}
```

