# Applying ViT-Adapter to WSDM Cup 2023 Toloka VQA Challenge

Our team wins the champion of [WSDM Cup 2023 Toloka VQA Challenge](https://codalab.lisn.upsaclay.fr/competitions/7434#learn_the_details).

The code is developed on top of [MMDetection v2.22.0](https://github.com/open-mmlab/mmdetection/tree/v2.22.0).

For details please see our [technical report]() (coming soon) for the competition.

If you use this code for a paper please cite:

```
# coming soon
@article{gao2023champion,
  title={Champion Solution for the WSDM2023 Toloka VQA Challenge},
  author={Gao, Shengyi and Chen, Zhe and Chen, Guo and Wang, Wenhai and Lu, Tong},
  journal={arXiv preprint arXiv:xxxx.xxxx},
  year={2023}
}
@article{chen2022vitadapter,
  title={Vision Transformer Adapter for Dense Predictions},
  author={Chen, Zhe and Duan, Yuchen and Wang, Wenhai and He, Junjun and Lu, Tong and Dai, Jifeng and Qiao, Yu},
  journal={arXiv preprint arXiv:2205.08534},
  year={2022}
}
```

## Usage

Install [MMDetection v2.22.0](https://github.com/open-mmlab/mmdetection/tree/v2.22.0).

```
# recommended environment: torch1.9 + cuda11.1
pip install torch==1.9.0+cu111 torchvision==0.10.0+cu111 torchaudio==0.9.0 -f https://download.pytorch.org/whl/torch_stable.html
pip install mmcv-full==1.4.2 -f https://download.openmmlab.com/mmcv/dist/cu111/torch1.9.0/index.html
pip install timm==0.4.12
pip install tfty
pip install mmdet==2.22.0 # for Mask2Former
pip install mmsegmentation==0.20.2
ln -s ../detection/ops ./
cd ops & sh make.sh # compile deformable attention
```

## Data Preparation

Preparing the [Toloka VQA Dataset](https://zenodo.org/record/7113781#.Y8tiVOxBz0o) and the [filtered GQA dataset]() (optional, coming soon).

```
wsdm2023
└── data
    ├── wsdm2023
    │   ├── annotations
    │   ├── train
    │   ├── train_sample
    │   └── test_public
    └── grounding_gqa
        ├── annotations
        └── images
```
## Sources

| Name          | Year | Type       | Data        | Repo                                                       | Paper                                                                                                                                                                           |
| :-------------: | :----: | :----------: | :-----------: | :----------------------------------------------------------: | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| Uni-Perceiver | 2022 | Supervised | Multi-Modal | [repo](https://github.com/fundamentalvision/Uni-Perceiver) | [paper](https://openaccess.thecvf.com/content/CVPR2022/papers/Zhu_Uni-Perceiver_Pre-Training_Unified_Architecture_for_Generic_Perception_for_Zero-Shot_and_CVPR_2022_paper.pdf) |

## Pre-training (GQA)

| Backbone      | Pre-train                                                                                                                                             | Head | Lr schd | Memory | Config                                                                | Download                                                                                                                                                                                                                                                        |
| :-------------: | :-----------------------------------------------------------------------------------------------------------------------------------------------------: | :----: | :-------: | :------: | :---------------------------------------------------------------------: | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| ViT-Adapter-B | [UniPerceiver-B](https://github.com/czczup/ViT-Adapter/releases/download/wsdm2023/uni-perceiver-base-L12-H768-224size-torch-pretrained_converted.pth) | DINO | 6ep     | 22G    | [config](./configs/dino_4scale_uniperceiver_adapter_base_6ep_gqa.py)  | [model](https://github.com/czczup/ViT-Adapter/releases/download/wsdm2023/dino_4scale_uniperceiver_adapter_base_6ep_gqa.pth) \| [log](https://github.com/czczup/ViT-Adapter/releases/download/wsdm2023/dino_4scale_uniperceiver_adapter_base_6ep_gqa.log.json)   |
| ViT-Adapter-L | [UniPerceiver-L](https://github.com/czczup/ViT-Adapter/releases/download/wsdm2023/uni-perceiver-large-L24-H1024-224size-pretrained_converted.pth)     | DINO | 6ep     | 30G    | [config](./configs/dino_4scale_uniperceiver_adapter_large_6ep_gqa.py) | [model](https://github.com/czczup/ViT-Adapter/releases/download/wsdm2023/dino_4scale_uniperceiver_adapter_large_6ep_gqa.pth) \| [log](https://github.com/czczup/ViT-Adapter/releases/download/wsdm2023/dino_4scale_uniperceiver_adapter_large_6ep_gqa.log.json) |

To pre-train the model on the filtered GQA Dataset on a single node with 8 gpus:

```shell
sh dist_train.sh configs/dino_4scale_uniperceiver_adapter_base_6ep_gqa.py 8
sh dist_train.sh configs/dino_4scale_uniperceiver_adapter_large_6ep_gqa.py 8
```

## Fine-tuning (Toloka VQA)

- We split a val set from the training set for offline model evaluation.

| Backbone      | Pre-train                                                                                                                                 | Head | Lr schd | Split    | Val | Public Test | Private Test | Mem. | Config                                                                                   | Download                                                                                                                                                                                                                                                                                         |
| :-------------: | :-----------------------------------------------------------------------------------------------------------------------------------------: | :----: | :-------: | :--------: | :---------: | :-----------------: | :------------------: | :------: | :----------------------------------------------------------------------------------------: | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| ViT-Adapter-B | [UniPerceiver-B+GQA](https://github.com/czczup/ViT-Adapter/releases/download/wsdm2023/dino_4scale_uniperceiver_adapter_base_6ep_gqa.pth)  | DINO | 24ep    | train    | 74.2      | 74.2              | -                  | 22G    | [config](./configs/dino_4scale_uniperceiver_adapter_base_24ep_gqa_wsdm2023.py)           | [model](https://github.com/czczup/ViT-Adapter/releases/download/wsdm2023/dino_4scale_uniperceiver_adapter_base_24ep_gqa_wsdm2023.pth) \| [log](https://github.com/czczup/ViT-Adapter/releases/download/wsdm2023/dino_4scale_uniperceiver_adapter_base_24ep_gqa_wsdm2023.log)                     |
| ViT-Adapter-L | [UniPerceiver-L+GQA](https://github.com/czczup/ViT-Adapter/releases/download/wsdm2023/dino_4scale_uniperceiver_adapter_large_6ep_gqa.pth) | DINO | 24ep    | train    | 76.7      | 76.9              | -                  | 30G    | [config](./configs/dino_4scale_uniperceiver_adapter_large_24ep_gqa_wsdm2023.py)          | [model](https://github.com/czczup/ViT-Adapter/releases/download/wsdm2023/dino_4scale_uniperceiver_adapter_large_24ep_gqa_wsdm2023.pth) \| [log](https://github.com/czczup/ViT-Adapter/releases/download/wsdm2023/dino_4scale_uniperceiver_adapter_large_24ep_gqa_wsdm2023.log)                   |
| ViT-Adapter-L | [UniPerceiver-L+GQA](https://github.com/czczup/ViT-Adapter/releases/download/wsdm2023/dino_4scale_uniperceiver_adapter_large_6ep_gqa.pth) | DINO | 24ep    | trainval | -         | **77.5**          | **76.347**         | 30G    | [config](./configs/dino_4scale_uniperceiver_adapter_large_24ep_gqa_wsdm2023_trainval.py) | [model](https://github.com/czczup/ViT-Adapter/releases/download/wsdm2023/dino_4scale_uniperceiver_adapter_large_24ep_gqa_wsdm2023_trainval.pth) \| [log](https://github.com/czczup/ViT-Adapter/releases/download/wsdm2023/dino_4scale_uniperceiver_adapter_large_24ep_gqa_wsdm2023_trainval.log) |

To train the model on the Toloka VQA Dataset on a single node with 8 gpus:

```shell
sh dist_train.sh configs/dino_4scale_uniperceiver_adapter_base_24ep_gqa_wsdm2023.py 8
sh dist_train.sh configs/dino_4scale_uniperceiver_adapter_large_24ep_gqa_wsdm2023.py 8
sh dist_train.sh configs/dino_4scale_uniperceiver_adapter_large_24ep_gqa_wsdm2023_trainval.py 8
```

## Evaluation

To evaluate our model on the val set on a single node with 8 gpus:

```shell
sh dist_test.sh /path/to/config /path/to/checkpoint 8 --eval bbox IoU
```

