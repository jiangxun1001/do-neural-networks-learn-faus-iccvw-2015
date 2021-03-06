# Running the CK+ experiments

# Contents
+ [Introduction](#introduction)
+ [Folder contents](#folder-contents)
+ [Running experiments](#running-experiments)

# Introduction

This folder contains the code used to obtain our results on the [CK+][CK+] dataset. Our experiments incorporate 
two particular types of regularization, they are:

+ a = Data Augmentation
+ d = Dropout

We will first describe the contents of this folder, and then walk you through
how to run the experiments.

## Folder contents
The folder contains:
``` shell
/cnn/train.py
/cnn_a/train.py
/cnn_d/train.py
/cnn_ad/train.py
ck_plus_checkpoint_checker.py
ck_plus_single_checkpoint_evaluator.py
data_paths.py
model.py
```

#### `train.py`
As you can see, there are several `train.py` files. Each one trains a cnn model with or without regularization.
Basically the `train.py` files do all the heavy lifting of running the individual experiments. They output a 
directory of model checkpoint files, and a log of the training process.

#### `ck_plus_single_checkpoint_evaluator.py`
This file contains a script that outputs the performance of a single save checkpoint.

#### `ck_plus_checkpoint_checker.py`
This file contains a script that examines all the checkpoints created by a 
single experiment, and chooses the best one.

#### `data_paths.py`
This file contains a single variable (``ck_plus_data_path``) which indicates the path to load the CK+ ``.npy`` files 
created when running the ``make_ck_plus_dataset.py`` file.

#### `model.py`
This file contains the CNN model used by the ``train.py`` files in each of our experiments.


# Running experiments

## CNN Training

You are now ready to train one of the four CNNs. The four folders starting with `cnn_` 
each contain a `train.py` file which will train the cnn subject to the 
regularizations described in the folder's suffix. 

### How to train a regular CNN

For example, `cnn` will train a cnn from a random initialization with no additional regularization.

You can train the cnn with following command: 
``` shell
# Snippet: cnn training
$ THEANO_FLAGS='floatX=float32,device=gpu0,nvcc.fastmath=True' \ 
python -u train.py --split 0  \ 
>& log0.txt & 
```

Since the [CK+][CK+] dataset is typically broken into splits and their results averaged,
the `--split` option indicates which of the 10 splits (0-9) will be used for testing. For example, 
the command above will train a net on folds 1-9 and evaluate the results on fold 0. The code 
will save the `.pkl` file containing the network parameters to a directory called `./checkpoints_0/` 
which will denote the split used.


### How to evaluate a model's performance

After you have trained a split to completion, you can find the best performing
checkpoint by running the checkpoint evaluator found in 
`ck_plus_checkpoint_checker.py`. We will use the model trained in `cnn` as an 
example. Simply run the following command:

``` shell
# Snippet: cnn checkpoint evaluation
$ THEANO_FLAGS='floatX=float32,device=gpu0,nvcc.fastmath=True' \ 
python -u ck_plus_checkpoint_checker.py --split 0 ./cnn/checkpoints_0/ \
>& cnn_best_performance_split_0.txt &
```

With this command, `ck_plus_checkpoint_checker.py` will iterate over the list of
checkpoints found in `./cnn/checkpoints_0/` and compute the accuracy on 
the test set. It will then select the checkpoint that yielded the highest
accuracy. The command also writes all of the results to a text file called 
`cnn_best_performance_split_0.txt`. Please remember, the `--split` argument
indicates which fold is being used as test/evaluation set.

### How to train the rest of the cnns

Now, if you want to train a network with specific regularizations active, 
the process is very simple. 

1. Using the legend above, create a suffix string (S) that corresponds to the 
   regularizations you wish to impose. 
2. Go to the `./cnn_S/` folder.
3. Run the `train.py` file as shown in the CNN Training section.


[CK+]:http://www.pitt.edu/~emotion/ck-spread.htm
