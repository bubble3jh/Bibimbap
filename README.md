<h1 align="center">Bibimbap: Pre-trained Models Ensemble<br>for Domain Generalization</h1>

<p align="center">
  <b>Pattern Recognition, 2024</b><br>
  Jinho Kang, Taero Kim, Yewon Kim, Changdae Oh, Jiyoung Jung, Rakwoo Chang, Kyungwoo Song
</p>

<p align="center">
  <a href="https://www.sciencedirect.com/science/article/pii/S0031320324001420"><img src="https://img.shields.io/badge/Paper-ScienceDirect-F36D21" alt="Paper"></a>
  <a href="https://doi.org/10.1016/j.patcog.2024.110391"><img src="https://img.shields.io/badge/DOI-10.1016%2Fj.patcog.2024.110391-0085CA" alt="DOI"></a>
  <img src="https://img.shields.io/badge/python-3.8-3776AB?logo=python&logoColor=white" alt="Python 3.8">
  <img src="https://img.shields.io/badge/PyTorch-EE4C2C?logo=pytorch&logoColor=white" alt="PyTorch">
</p>

<p align="center">
  <img src="figures/flowchart.png" width="90%" alt="Flowchart of the Model Bibimbap ensemble framework">
</p>

Official PyTorch implementation of **Model Bibimbap**, a framework that ensembles the weights of fine-tuned
models and fine-tunes the ensemble once more for robustness under domain shift.

- **Why naive weight ensembling underfits.** Under extreme domain shift, the ensembled models may not lie in the
  same loss basin, and a loss barrier can separate them.
- **Fine-tune after ensembling.** A fine-tuning step after the weighted ensemble resolves the underfitting caused
  by the loss barrier and stabilizes the batch normalization statistics.
- **Diversity matters.** The diversity of the ensembled models affects domain generalization.
- **Validated on images and molecules.** ImageNet-1K and chemical molecule data, whose splits create strong
  domain shift.

## Motivation

Weight ensembles such as [Model Ratatouille](https://github.com/facebookresearch/ModelRatatouille) assume that the
models to be averaged share a loss basin. When the domains differ strongly this may not hold, and the averaged
model can land on a loss barrier.

<p align="center">
  <img src="figures/illustration.png" width="80%" alt="Toy illustration of an ensembled model on a loss barrier">
</p>

<p align="center"><em>Fig. 1. Under extreme domain shift, the ensembled model can sit on a loss barrier and needs
additional fine-tuning.</em></p>

<p align="center">
  <img src="figures/loss_interpolation.png" width="60%" alt="Loss interpolation between models on chemical data">
</p>

<p align="center"><em>Fig. 2. Loss interpolation on chemical data shows a loss barrier between models, so mode
connectivity is not guaranteed.</em></p>

## Results

<p align="center">
  <img src="figures/auroc.png" width="60%" alt="AUROC of Bibimbap with and without the final fine-tuning">
</p>

<p align="center"><em>Fig. 3. AUROC during training (top) and testing (bottom). Transparent curves are the auxiliary
models of the ensemble.</em></p>

Without the final fine-tuning, Bibimbap starts with a much lower training AUROC, a sign of underfitting. With it,
both the training and the test AUROC rise considerably, which shows that the fine-tuning overcomes the loss barrier.

## Getting started

### Installation

Tested with Python 3.8.15.

```
pytorch                   1.1.0
torch-cluster             1.6.0
torch-geometric           2.2.0
torch-scatter             2.0.9
torch-sparse              0.6.13
torch-spline-conv         1.2.1
rdkit                     2022.9.3
tqdm                      4.62.3
tensorboardx              2.5.1
```

### Data

Download the [chemistry datasets](http://snap.stanford.edu/gnn-pretrain/data/chem_dataset.zip) (2.5 GB), unzip
them, and place them under `dataset/`.

### Running

Model Bibimbap builds its ensemble in four steps. `bibimbap.sh` runs the whole pipeline.

**1. Fine-tune a pre-trained model.** `model_gin/` holds models pre-trained in different ways.

```bash
python finetune.py --input_model_file PRETRAINED_MODEL_PATH --filename OUTPUT_FILENAME --dataset DATASET --epochs EPOCHS
```

Add `--freeze_bn` to freeze the batch normalization layers. This step gives one fine-tuned model per dataset.

**2. Linear probing on the target dataset.**

```bash
python finetune.py --input_model_file PRETRAINED_MODEL_PATH --filename OUTPUT_FILENAME --dataset DATASET --epochs EPOCHS --freeze_gnn
```

The resulting linear classifier replaces the classifier of the auxiliary models in the next step.

**3. Auxiliary fine-tuning.**

```bash
python finetune.py --input_model_file FINETUNED_MODEL_PATH --replace_classifier LP_MODEL_PATH --dataset DATASET --filename OUTPUT_FILENAME --epochs EPOCHS
```

`--freeze_bn` is available here as well. We name the outputs `target_${target}_aux_${aux}_${model_ver}`, where
`target` is the target dataset, `aux` the auxiliary dataset, and `model_ver` a string such as `fr_nfr` that records
whether batch normalization was frozen in the previous and the current step.

**4. Ensemble and final fine-tuning.**

```bash
python finetune.py --averaging_target DATASET --dataset DATASET --averaging_aux AUX_MODELS --epochs EPOCHS --model_ver MODEL_VER --ensemble_method uniform
```

`DATASET` is the target dataset and `AUX_MODELS` are the names of the auxiliary models. `--freeze_gnn` (features),
`--freeze_lc` (classifier), and `--freeze_bn` can be combined to find the best model for each dataset. The ensemble
weights are uniform (`--ensemble_method uniform`), sampled from a Dirichlet distribution
(`--ensemble_method dirichlet`), or given by hand with `--ens_weight`.

## Citation

```bibtex
@article{KANG2024110391,
  title = {Bibimbap : Pre-trained models ensemble for Domain Generalization},
  journal = {Pattern Recognition},
  volume = {151},
  pages = {110391},
  year = {2024},
  issn = {0031-3203},
  doi = {https://doi.org/10.1016/j.patcog.2024.110391},
  url = {https://www.sciencedirect.com/science/article/pii/S0031320324001420},
  author = {Jinho Kang and Taero Kim and Yewon Kim and Changdae Oh and Jiyoung Jung and Rakwoo Chang and Kyungwoo Song}
}
```

The lab-maintained copy of this repository is at [MLAI-Yonsei/Bibimbap](https://github.com/MLAI-Yonsei/Bibimbap).
