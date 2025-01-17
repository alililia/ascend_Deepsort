# Contents

- [Contents](#contents)
    - [T-GCN Description](#t-gcn-description)
    - [Model Architecture](#model-architecture)
    - [Dataset](#dataset)
        - [Dataset used: SZ-taxi](#dataset-used-sz-taxi)
        - [Dataset used: Los-loop](#dataset-used-los-loop)
            - [Dataset organize way](#dataset-organize-way)
    - [Environment Requirements](#environment-requirements)
    - [Quick Start](#quick-start)
    - [Script Description](#script-description)
        - [Script and Sample Code](#script-and-sample-code)
        - [Parameter configuration](#parameter-configuration)
        - [Training Process](#training-process)
            - [Training](#training-1)
        - [Evaluation Process](#evaluation-process)
            - [Evaluation](#evaluation-1)
    - [Model Description](#model-description)
        - [Performance](#performance)
            - [Training Performance](#training-performance)
            - [Evaluation Performance](#evaluation-performance)
    - [Description of Random Situation](#description-of-random-situation)
    - [ModelZoo Homepage](#modelzoo-homepage)

## [T-GCN Description](#contents)

T-GCN is a combination of spatial convolution and GRU recurrent cell for prediction of car traffic characteristics.

[Paper](https://arxiv.org/pdf/1811.05320v3.pdf): Zhao L, Song Y, Zhang C, et al. T-GCN: A temporal graph convolutional network for traffic prediction. IEEE Transactions on Intelligent Transportation Systems, 2019, 21(9): 3848-3858.

## [Model Architecture](#contents)

Graph convolution network is used for feature encoding of spatial dependence of car traffic parameters for given time frame and consists of 2 convolution layers with sigmoid and ReLU activations. Time sequences of encoded spatial features are used in GRU to predict future spatial distribution of car traffic parameters for a given prediction horizon.

## [Dataset](#contents)

Note that you can run the scripts based on the dataset mentioned in original paper or widely used in relevant domain/network architecture. In the following sections, we will introduce how to run the scripts using the related dataset below.

### Dataset used: [SZ-taxi](<https://github.com/lehaifeng/T-GCN/tree/master/T-GCN/T-GCN-PyTorch/data>)

This dataset contains the taxi trajectory of Shenzhen from Jan. 1 to Jan. 31, 2015. Car traffic speeds were recorded for 156 roads and 2976 time frames.

### Dataset used: [Los-loop](https://github.com/lehaifeng/T-GCN/tree/master/T-GCN/T-GCN-PyTorch/data)

This dataset was collected in the highway of Los Angeles County in real time by loop detectors. It contains traffic speed for 207 roads and 2016 time frames.

For both datasets, the first 80% of time frames is  used for training, the rest of data is used for validation. All input data is normalized to interval [0, 1]

#### Dataset organize way

```python
.
└─tgcn
  ├─data
    ├─SZ-taxi
        ├─adj.csv
        └─feature.csv
    ├─Los-loop
        ├─adj.csv
        └─feature.csv
...
```

## [Environment Requirements](#contents)

- Framework
    - [MindSpore](https://www.mindspore.cn/install/en)
- For more information, please check the resources below：
    - [MindSpore Tutorials](https://www.mindspore.cn/tutorials/en/master/index.html)
    - [MindSpore Python API](https://www.mindspore.cn/docs/api/en/master/index.html)

## [Quick Start](#contents)

After installing MindSpore via the official website, choose dataset and set the datasets path in `src/config.py` file.
You can start training and evaluation as follows:

- Training


```python
bash ./scripts/run_standalone_train.sh [DEVICE_ID]

```

Example：

  ```python
  bash ./scripts/run_standalone_train.sh 0

  ```

- Evaluation：

```python
bash ./scripts/run_eval.sh [DEVICE_ID]
```

Example：

  ```python
  bash ./scripts/run_eval.sh 0
  ```

## [Script Description](#contents)

### [Script and Sample Code](#contents)

```python
|-- README.md                                      # English README
|-- README_CN.md                                   # Chinese README
|-- ascend310_infer                                # Ascend 310 inference source coode
|-- eval.py                                        # Evaluate
|-- export.py                                      # MINDIR model export
|-- preprocess.py                                  # Ascend 310 preprocess
|-- postprocess.py                                 # Ascend 310 postprocess
|-- requirements.txt                               # pip dependencies
|-- scripts
|   |-- run_distributed_train_ascend.sh            # Ascend distributed training script
|   |-- run_eval.sh                                # Evaluation script
|   |-- run_export.sh                              # MINDIR model export script
|   |-- run_infer_310.sh                           # Ascend 310 inference script
|   `-- run_standalone_train.sh                    # Single-device training script
|-- src
|   |-- __init__.py
|   |-- callback.py                                # Custom callback functions
|   |-- config.py                                  # Configuration file
|   |-- dataprocess.py                             # Data preprocessing functions
|   |-- metrics.py                                 # Custom metrics functions
|   |-- model
|   |   |-- __init__.py
|   |   |-- graph_conv.py                          # Graph convolution module
|   |   |-- loss.py                                # Loss function module
|   |   `-- tgcn.py                                # T-GCN model architecture
|   `-- task.py                                    # Supervised forecast task
`-- train.py                                       # Training

```

### [Parameter configuration](#contents)

Parameters for both training and evaluation can be set in `src/config.py`.

```python
class ConfigTGCN:
    device = 'Ascend'
    seed = 1
    data_path = '/path/to/data'
    dataset = 'SZ-taxi'
    hidden_dim = 100
    seq_len = 4
    pre_len = 1
    train_split_rate = 0.8
    epochs = 3000
    batch_size = 64
    learning_rate = 0.001
    weight_decay = 1.5e-3
    data_sink = True
```

### [Training Process](#contents)

#### Training



- Training using single device (1p)

```python
bash ./scripts/run_standalone_train.sh 0
```

- Distributed Training (8p)

```python
```

`save_best` key in config file can be set to `True` for automatic evaluation of the model on the evaluation set. In that case, checkpoint with best RMSE will be saved. Otherwise, checkpoint will be saved after each epoch. Automatic evaluation will drastically increase training time.
Checkpoints will be saved in `./checkpoints/` folder. Checkpoint filename format: `[DATASET_NAME]_[PRE_LEN].ckpt`.

### [Evaluation Process](#contents)

#### Evaluation

Evaluation script uses checkpoint file with `[DATASET_NAME]` and `[PRE_LEN]` specified in `./src/config.py`. To start evaluation, run the following command:

```python
bash ./scripts/run_eval.sh [DEVICE_ID]
# Example:
bash ./scripts/run_eval.sh 0
```

## [Model Description](#contents)

### [Performance](#contents)

#### Training Performance

Training performance in the following tables is obtained by the T-GCN model based on the SZ-taxi dataset, which predicts the traffic speed of the next 15 minutes, 30 minutes, 45 minutes, and 60 minutes (pre_len 1, 2, 3, 4 respectively), corresponding measurements are the average of 4 sets of training tasks. Loss values are given for separate pre_len, and averaged within last 50 entries in corresponding log files.



## [Description of Random Situation](#contents)

Global training random seed is fixed in `./train.py` with `mindspore.set_seed()` (default 1) and can be modified in the `./src/config.py`.

## [ModelZoo Homepage](#contents)  

Please check the official [homepage](https://gitee.com/mindspore/models).  
