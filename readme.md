# Face Alignment in Full Pose Range: A 3D Total Solution

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

**\[Updates\]**

 - `2018.11.11`: **Update inference pipeline: infer/serialize 3D face shape and 68 landmarks given one arbitrary image, please see readme.md below for more details.**
 - `2018.11.9`: Update trained model with higher performance in [models](./models).
 - `2018.11.9`: Add removed-neck version of BFM model in [BFM_Remove_Neck](./BFM_Remove_Neck).
 - `2018.10.4`: Add Matlab face mesh rendering demo in [visualize](./visualize).
 - `2018.9.9`: Add pre-process of face cropping in [benchmark](./benchmark).

## Introduction
This repo holds the pytorch improved re-implementation of paper [Face Alignment in Full Pose Range: A 3D Total Solution](https://arxiv.org/abs/1804.01005). Several additional works are added in this repo, including real-time training, training strategy and so on. Therefore, this repo is far more than re-implementation. One related blog will be published for some important technique details in future.
As far, this repo releases the pre-trained first-stage pytorch models of MobileNet-V1 structure, the training dataset and code.
And the inference time is about **0.27ms per image** (input batch with 128 iamges) on GeForce GTX TITAN X.
**These repo will keep being updated, thus any meaningful issues and PR are welcomed.**

Several results (inferenced from model *phase1_wpdc_vdc.pth.tar*) are shown below.
<p align="center">
  <img src="imgs/landmark_3d.jpg" alt="Landmark 3D" width="1000px">
</p>

<p align="center">
  <img src="imgs/vertex_3d.jpg" alt="Vertex 3D" width="750px">
</p>

## Getting started
### Requirements
 - PyTorch >= 0.4.1
 - Python >= 3.6 (Numpy, Scipy, Matplotlib)
 - Dlib (Dlib is used for detecting face and landmarks. There is no need to use Dlib if you can provide face bouding bbox and landmarks. Optionally, you can use the two-step inference strategy without initialized landmarks.)
 - OpenCV (Python version, for image IO opertations.)

In addition, I strongly recommend using Python3.6+ instead of older version for its better design.

### Usage

1. Clone this repo (this may take some time as it is a little big)
    ```
    git clone https://github.com/cleardusk/3DDFA.git  # or git@github.com:cleardusk/3DDFA.git
    cd 3DDFA
    ```
2. Run the `main.py` with arbitrary image as input
    ```
    python3 main.py -f samples/test1.jpg
    ```
    If you can see these outputs in terminal, you run it successfully.
    ```
    Dump tp samples/test1_0.ply
    Dump tp samples/test1_0.mat
    Dump tp samples/test1_1.ply
    Dump tp samples/test1_1.mat
    Save visualization result to samples/test1_3DDFA.jpg
    ```

    Because `test1.jpg` has two faces, there are two `mat` (stores dense face vertices, can be rendered by Matlab, see [visualize](./visualize)) and `ply` files (can be rendered by Meshlab or Microsoft 3D Builder) predicted.

    Please run `python3 main.py -h` or review the code for more details.

    The result `samples/test1_3DDFA.jpg` is shown below

<p align="center">
  <img src="samples/test1_3DDFA.jpg" alt="samples" width="700px">
</p>

    One more example:

    ```
    python3 ./main.py -f samples/emma_input.jpg --box_init=two --dlib_bbox=false
    ```
    and the result `samples/emma_input_3DDFA.jpg` is shown below

<p align="center">
  <img src="samples/emma_input_3DDFA.jpg" alt="samples" width="700px">
</p>



## Citation
    @article{zhu2017face,
      title={Face Alignment in Full Pose Range: A 3D Total Solution},
      author={Zhu, Xiangyu and Lei, Zhen and Li, Stan Z and others},
      journal={IEEE Transactions on Pattern Analysis and Machine Intelligence},
      year={2017},
      publisher={IEEE}
    }
    


## Inference speed
When batch size is 128, the inference time of MobileNet-V1 takes about 34.7ms. The average speed is about **0.27ms/pic**.

<p align="left">
  <img src="imgs/inference_speed.png" alt="Inference speed" width="600px">
</p>

## Evaluation
First, you should download the cropped testset ALFW and ALFW-2000-3D in [test.data.zip](https://pan.baidu.com/s/1DTVGCG5k0jjjhOc8GcSLOw), then unzip it and put it in the root directory.
Next, run the benchmark code by providing trained model path.
I have already provided four pre-trained models in `models` directory. These models are trained using different loss in the first stage. The model size is about 13M due to the high efficiency of MobileNet-V1 structure.
```
python3 ./benchmark.py -c models/phase1_wpdc_vdc.pth.tar
```

The performances of pre-trained models are shown below. In the first stage, the effectiveness of different loss is in order: WPDC > VDC > PDC. While the strategy using VDC to finetune WPDC achieves the best result.

| Model | AFLW (21 pts) | AFLW 2000-3D (68 pts) |
|:-:|:-:|:-:|
| *phase1_pdc.pth.tar*  | 6.956±0.981 | 5.644±1.323 |
| *phase1_vdc.pth.tar*  | 6.717±0.924 | 5.030±1.044 |
| *phase1_wpdc.pth.tar* | 6.348±0.929 | 4.759±0.996 |
| *phase1_wpdc_vdc.pth.tar* | **5.401±0.754** | **4.252±0.976** |
| *phase1_wpdc_vdc_v2.pth.tar* \[newly add\] | **5.298±0.776** | **4.090±0.964** |

## Training
The training scripts lie in `training` directory. The related resources are in below table.

| Data | Link | Description |
|:-:|:-:|:-:|
| train.configs | [BaiduYun](https://pan.baidu.com/s/1ozZVs26-xE49sF7nystrKQ) or [Google Drive](https://drive.google.com/open?id=1dzwQNZNMppFVShLYoLEfU3EOj3tCeXOD), 217M | The directory contraining 3DMM params and filelists of training dataset |
| train_aug_120x120.zip | [BaiduYun](https://pan.baidu.com/s/19QNGst2E1pRKL7Dtx_L1MA) or [Google Drive](https://drive.google.com/open?id=17LfvBZFAeXt0ACPnVckfdrLTMHUpIQqE), 2.15G | The cropped images of augmentation training dataset |
| test.data.zip | [BaiduYun](https://pan.baidu.com/s/1DTVGCG5k0jjjhOc8GcSLOw) or [Google Drive](https://drive.google.com/file/d/1r_ciJ1M0BSRTwndIBt42GlPFRv6CvvEP/view?usp=sharing), 151M | The cropped images of AFLW and ALFW-2000-3D testset |

After preparing the training dataset and configuration files, go into `training` directory and run the bash scripts to train. 
The training parameters are all presented in bash scripts.

## FQA
1. Face bounding box initialization

    The original paper validates that using detected bounding box instead of ground truth box will cause a little performance drop. Thus the current face cropping method is robustest. Quantitative results are shown in below table.

<p align="center">
  <img src="imgs/bouding_box_init.png" alt="bounding box" width="500px">
</p>

## Acknowledgement
Thanks for [Xiangyu Zhu](http://www.cbsr.ia.ac.cn/users/xiangyuzhu/)'s great work.
Thanks for your interest in this repo.
If your research benefits from this repo, please cite it and star it 😃