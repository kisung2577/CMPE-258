**Installation**

Installing PyTorch:
$ conda install pytorch=0.4.1 cuda90 -c pytorch

Install dependencies:
$ pip install -r requirements.txt

**Pretrained model**

Download ingredient and instruction vocabularies [here](https://dl.fbaipublicfiles.com/inversecooking/ingr_vocab.pkl) and [here](https://dl.fbaipublicfiles.com/inversecooking/instr_vocab.pkl), respectively.

Download pretrained model [here](https://dl.fbaipublicfiles.com/inversecooking/modelbest.ckpt).

**Demo**

You can use my pretrained model to get recipes for your images.

Download the required files (listed above), place them under the data directory, and try my demo notebook src/demo.ipynb.

Note: The demo will run on GPU if a device is found, else it will use CPU.

**Data**

Download [Recipe1M](http://im2recipe.csail.mit.edu/dataset/download) (registration required)

Extract files somewhere (we refer to this path as path_to_dataset).

The contents of path_to_dataset should be the following:

det_ingrs.json
layer1.json
layer2.json
images/
images/train
images/val
images/test

Note: all python calls below must be run from ./src

**Build vocabularies**

$ python build_vocab.py --recipe1m_path path_to_dataset

**Images to LMDB (Optional, but recommended)**

For fast loading during training:

$ python utils/ims2file.py --recipe1m_path path_to_dataset

If you decide not to create this file, use the flag --load_jpeg when training the model.

**Traning**

Create a directory to store checkpoints for all models you train (e.g. ../checkpoints and point --save_dir to it.)

I train my model in two stages:

1. Ingredient prediction from images

python train.py --model_name im2ingr --batch_size 150 --finetune_after 0 --ingrs_only \
--es_metric iou_sample --loss_weight 0 1000.0 1.0 1.0 \
--learning_rate 1e-4 --scale_learning_rate_cnn 1.0 \
--save_dir ../checkpoints --recipe1m_dir path_to_dataset


2. Recipe generation from images and ingredients (loading from 1.)

python train.py --model_name model --batch_size 256 --recipe_only --transfer_from im2ingr \
--save_dir ../checkpoints --recipe1m_dir path_to_dataset

Check training progress with Tensorboard from ../checkpoints:

$ tensorboard --logdir='../tb_logs' --port=6006

**Evaluation**

Save generated recipes to disk with python sample.py --model_name model --save_dir ../checkpoints --recipe1m_dir path_to_dataset --greedy --eval_split test.

This script will return ingredient metrics (F1 and IoU)

**License**

inversecooking is released under MIT license, see [LICENSE]() for details.
