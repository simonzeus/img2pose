# img2pose: Face Alignment and Detection via 6DoF, Face Pose Estimation

[![License: CC BY-NC 4.0](https://img.shields.io/badge/License-CC%20BY--NC%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc/4.0/)

<figure>
  <img src="./teaser.jpeg" style="width:100%">
  <figcaption>Figure 1: We estimate the 6DoF rigid transformation of a 3D face (rendered in silver), aligning it with even the tiniest faces, without face detection or facial landmark localization. Our estimated 3D face locations are rendered by descending distances from the camera, for coherent visualization.</figcaption>
</figure>

## Table of contents

<!--ts-->
- [Paper details](#paper-details)
  * [arXiv preprint](#arxiv-preprint)
  * [Abstract](#abstract)
  * [Authors](#authors)
  * [Citation](#citation)
- [Installation](#installation)
- [Training](#training)
  * [Prepare WIDER FACE dataset](#prepare-wider-face-dataset)
  * [Train](#train)
- [Testing](#testing)
  * [WIDER FACE dataset evaluation](#wider-face-dataset-evaluation)
  * [Visualizing trained model](#visualizing-trained-model)
  * [AFLW2000-3D dataset evaluation](#aflw2000-3d-dataset-evaluation)
  * [BIWI dataset evaluation](#biwi-dataset-evaluation)
  * [Testing on your own images](#testing-on-your-own-images)
- [Resources](#resources)
- [Citation](#citation)
- [License](#license)
<!--te-->

## Paper details

### [arXiv preprint](https://arxiv.org/abs/2012.07791)

### Abstract
> We propose real-time, six degrees of freedom (6DoF), 3D face pose estimation without face detection or landmark localization. We observe that estimating the 6DoF rigid transformation of a face is a simpler problem than facial landmark detection, often used for 3D face alignment. In addition, 6DoF offers more information than face bounding box labels. We leverage these observations to make multiple contributions: (a) We describe an easily trained, efficient, Faster R-CNN--based model which regresses 6DoF pose for all faces in the photo, without preliminary face detection. (b) We explain how pose is converted and kept consistent between the input photo and arbitrary crops created while training and evaluating our model. (c) Finally, we show how face poses can replace detection bounding box training labels. Tests on AFLW2000-3D and BIWI show that our method runs at real-time and outperforms state of the art (SotA) face pose estimators. Remarkably, our method also surpasses SotA models of comparable complexity on the WIDER FACE detection benchmark, despite not been optimized on bounding box labels.

### Authors
Vítor Albiero, Xingyu Chen, Xi Yin, Guan Pang, Tal Hassner

### Citation
If you use any part of our code, please cite our paper.
```
@InProceedings{img2pose,
    author = {Vítor Albiero, Xingyu Chen, Xi Yin, Guan Pang, Tal Hassner},
    title = {img2pose: Face Alignment and Detection via 6DoF, Face Pose Estimation},
    booktitle = {arxiv},
    year = {2020}
}
```

## Installation
Install dependecies.
```
pip install -r requirements.txt
```
Install the renderer (used to visualize predictions).
```
cd Sim3DR
sh build_sim3dr.sh
```

## Training
### Prepare WIDER FACE dataset
First, download our annotations as instructed in [Annotations](https://github.com/vitoralbiero/img2pose/wiki/Annotations).

Download [WIDER FACE](http://shuoyang1213.me/WIDERFACE/) dataset and extract to datasets/WIDER_Face.

Then, to create the train and validation files (LMDB), run the following scripts.

```
python3 convert_json_list_to_lmdb.py
--json_list ./annotations/WIDER_train_annotations.txt
--dataset_path ./datasets/WIDER_Face/WIDER_train/images/
--dest ./datasets/lmdb/
-—train
```
This first script will generate a LMDB dataset, which contains the training images along with annotations. It will also output a pose mean and std deviation files, which will be used for training and testing.
```
python3 convert_json_list_to_lmdb.py 
--json_list ./annotations/WIDER_val_annotations.txt 
--dataset_path ./datasets/WIDER_Face/WIDER_val/images/ 
--dest ./datasets/lmdb
```
This second script will create a LMDB containing the validation images along with annotations.

### Train
Once the LMDB train/val files are created, to start training simple run the script below.
```
CUDA_VISIBLE_DEVICES=0 python3 train.py
--pose_mean ./datasets/lmdb/WIDER_train_annotations_pose_mean.npy
--pose_stddev ./datasets/lmdb/WIDER_train_annotations_pose_stddev.npy
--workspace ./workspace/
--train_source ./datasets/lmdb/WIDER_train_annotations.lmdb
--val_source ./datasets/lmdb/WIDER_val_annotations.lmdb
--prefix trial_1
--batch_size 2
--lr_plateau
--early_stop
--random_flip
--random_crop
--max_size 1400
```
For now, only single GPU training is tested. Distributed training is partially implemented, PRs welcome.

## Testing
To evaluate with the pretrained model, download the model from [Model Zoo](https://github.com/vitoralbiero/img2pose/wiki/Model-Zoo), and extract it to the main folder. It will create a folder called models, which contains the model weights and the pose mean and std dev that was used for training.

If evaluating with own trained model, change the pose mean and standard deviation to the ones trained with.

### WIDER FACE dataset evaluation
If you haven't done already, download the [WIDER FACE](http://shuoyang1213.me/WIDERFACE/) dataset and extract to datasets/WIDER_Face.

```
python3 evaluation/evaluate_wider.py 
--dataset_path datasets/WIDER_Face/WIDER_val/images/
--dataset_list datasets/WIDER_Face/wider_face_split/wider_face_val_bbx_gt.txt
--pretrained_path models/img2pose_v1.pth
--output_path results/WIDER_FACE/Val/
```

To check mAP and plot curves, download the [eval tools](http://shuoyang1213.me/WIDERFACE/) and point to results/WIDER_FACE/Val.

### Visualizing trained model
To visualize a trained model on the WIDER FACE validation set run the notebook [visualize_trained_model_predictions](evaluation/jupyter_notebooks/visualize_trained_model_predictions.ipynb).

### AFLW2000-3D dataset evaluation
Download the [AFLW2000-3D](http://www.cbsr.ia.ac.cn/users/xiangyuzhu/projects/3DDFA/Database/AFLW2000-3D.zip) dataset and unzip to datasets/AFLW2000.

Run the notebook [aflw_2000_3d_evaluation](./evaluation/jupyter_notebooks/aflw_2000_3d_evaluation.ipynb).

### BIWI dataset evaluation
Download the [BIWI](http://data.vision.ee.ethz.ch/cvl/gfanelli/kinect_head_pose_db.tgz) dataset and unzip to datasets/BIWI.

Run the notebook [biwi_evaluation](./evaluation/jupyter_notebooks/biwi_evaluation.ipynb).

### Testing on your own images

Run the notebook [test_own_images](./evaluation/jupyter_notebooks/test_own_images.ipynb).

## Resources
[Model Zoo](https://github.com/vitoralbiero/img2pose/wiki/Model-Zoo)

[Annotations](https://github.com/vitoralbiero/img2pose/wiki/Annotations)

[Data Zoo](https://github.com/vitoralbiero/img2pose/wiki/Data-Zoo)

## License
Check [license](./license.md) for license details.
