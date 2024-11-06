# WHAM: 正確な3D動作を伴う世界座標系での人体再構成

<a href="https://pytorch.org/get-started/locally/"><img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-ee4c2c?logo=pytorch&logoColor=white"></a> [![report](https://img.shields.io/badge/arxiv-report-red)](https://arxiv.org/abs/2312.07531) <a href="https://wham.is.tue.mpg.de/"><img alt="Project" src="https://img.shields.io/badge/-Project%20Page-lightgrey?logo=Google%20Chrome&color=informational&logoColor=white"></a> [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1ysUtGSwidTQIdBQRhq0hj63KbseFujkn?usp=sharing)
 [![PWC](https://img.shields.io/endpoint.svg?url=https://paperswithcode.com/badge/wham-reconstructing-world-grounded-humans/3d-human-pose-estimation-on-3dpw)](https://paperswithcode.com/sota/3d-human-pose-estimation-on-3dpw?p=wham-reconstructing-world-grounded-humans) [![PWC](https://img.shields.io/endpoint.svg?url=https://paperswithcode.com/badge/wham-reconstructing-world-grounded-humans/3d-human-pose-estimation-on-emdb)](https://paperswithcode.com/sota/3d-human-pose-estimation-on-emdb?p=wham-reconstructing-world-grounded-humans)

## はじめに
このリポジトリは[WHAM: Reconstructing World-grounded Humans with Accurate 3D Motion](https://arxiv.org/abs/2312.07531)の公式[PyTorch](https://pytorch.org/)実装です。詳細については[プロジェクトページ](https://wham.is.tue.mpg.de/)をご覧ください。

## インストール
詳細については[Installation](docs/INSTALL.md)をご参照ください。

## クイックデモ

### [<img src="https://i.imgur.com/QCojoJk.png" width="30"> WHAMデモのGoogle Colabが利用可能になりました](https://colab.research.google.com/drive/1ysUtGSwidTQIdBQRhq0hj63KbseFujkn?usp=sharing)

### 登録

SMPL体型モデル（Neutral、Female、Male）をダウンロードするには、[SMPL](https://smpl.is.tue.mpg.de/)と[SMPLify](https://smplify.is.tue.mpg.de/)に登録する必要があります。デモデータの取得時に両方のホームページのユーザー名とパスワードが使用されます。

次に、以下のスクリプトを実行してデモデータを取得します。このスクリプトは、学習済みモデルやデモ動画を含む必要な依存関係をすべてダウンロードします。

```bash
bash fetch_demo_data.sh
```

サンプル動画で試すことができます：
```
python demo.py --video examples/IMG_9732.mov --visualize
```

カメラの焦点距離は[CLIFF](https://github.com/haofanwang/CLIFF)に従って想定しています。以下のデモ例のように、SLAMのための既知のカメラ内部パラメータ[fx fy cx cy]を指定できます：
```
python demo.py --video examples/drone_video.mp4 --calib examples/drone_calib.txt --visualize
```

カメラ座標系での動きのみを取得したい場合は、SLAMをスキップできます：
```
python demo.py --video examples/IMG_9732.mov --visualize --estimate_local_only
```

後処理として Temporal SMPLifyを使用してWHAMの結果をさらに改善することができます。これにより、2D位置合わせと3D精度が向上します。デモを実行する際に`--run_smplify`フラグを追加するだけです。

## Docker

詳細については[Docker](docs/DOCKER.md)をご参照ください。

## Python API

詳細については[API](docs/API.md)をご参照ください。

## データセット
詳細については[Dataset](docs/DATASET.md)をご参照ください。

## 評価
```bash
# 3DPWデータセットでの評価
python -m lib.eval.evaluate_3dpw --cfg configs/yamls/demo.yaml TRAIN.CHECKPOINT checkpoints/wham_vit_w_3dpw.pth.tar

# RICHデータセットでの評価
python -m lib.eval.evaluate_rich --cfg configs/yamls/demo.yaml TRAIN.CHECKPOINT checkpoints/wham_vit_w_3dpw.pth.tar

# EMDBデータセットでの評価（W-MPJPEとWA-MPJPEも計算）
python -m lib.eval.evaluate_emdb --cfg configs/yamls/demo.yaml --eval-split 1 TRAIN.CHECKPOINT checkpoints/wham_vit_w_3dpw.pth.tar   # EMDB 1

python -m lib.eval.evaluate_emdb --cfg configs/yamls/demo.yaml --eval-split 2 TRAIN.CHECKPOINT checkpoints/wham_vit_w_3dpw.pth.tar   # EMDB 2
```

## トレーニング
WHAMのトレーニングは2つの異なるステージで構成されています：(1) AMASSデータセットを通じた2DからSMPLへのリフティング、(2) 動画データセットを使用した特徴統合による微調整。トレーニングデータセットの前処理については[Dataset](docs/DATASET.md)をご参照ください。

### ステージ1
```bash
python train.py --cfg configs/yamls/stage1.yaml
```

### ステージ2
ステージ2のトレーニングには、ステージ1の事前学習結果が必要です。あなたの事前学習結果を使用するか、[Google Drive](https://drive.google.com/file/d/1Erjkho7O0bnZFawarntICRUCroaKabRE/view?usp=sharing)から重みをダウンロードして`checkpoints/wham_stage1.tar.pth`として保存してください。
```bash
python train.py --cfg configs/yamls/stage2.yaml TRAIN.CHECKPOINT <PATH-TO-STAGE1-RESULTS>
```

### BEDLAMでのトレーニング
TBD

## 謝辞
議論とプルーフリーディングをしていただいたHongwei YiとSilvia Zuffiに心より感謝申し上げます。この研究の一部は、Soyong ShinがMax Planck Institute for Intelligence Systemでインターンをしている間に行われました。

基本的な実装は[VIBE](https://github.com/mkocabas/VIBE)と[TCMR](https://github.com/hongsukchoi/TCMR_RELEASE)から多く借用しています。2Dキーポイント検出には[ViTPose](https://github.com/ViTAE-Transformer/ViTPose)を、カメラモーションの抽出には[DPVO](https://github.com/princeton-vl/DPVO)、[DROID-SLAM](https://github.com/princeton-vl/DROID-SLAM)を使用しています。詳細については各公式ウェブサイトをご覧ください。

## TODO

- [ ] データの前処理

- [x] トレーニングの実装

- [x] Colabデモのリリース

- [x] カスタム動画用のデモ

## 引用
```
@InProceedings{shin2023wham,  
title={WHAM: Reconstructing World-grounded Humans with Accurate 3D Motion},
author={Shin, Soyong and Kim, Juyong and Halilaj, Eni and Black, Michael J.},  
booktitle={Computer Vision and Pattern Recognition (CVPR)},  
year={2024}  
}  
```

## ライセンス
詳細については[License](./LICENSE)をご参照ください。

## お問い合わせ
本研究に関するご質問は、soyongs@andrew.cmu.eduまでお願いいたします。