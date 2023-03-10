# PHONEix

This is the homepage for work PHONEix.

《PHONEix: Acoustic Feature Processing Strategy for Enhanced Singing Pronunciation with Phoneme Distribution Predictor》(Submit to ICASSP 2023) Paper will be released soon.

## Table of Contents

 * [Codes for PHONEix](#codes-for-phoneix)
   * [Installation](#installation)
   * [How to run](#how-to-run)
   * [Evaluation](#evaluation)
 * [Codes for baselines](#codes-for-baselines)
   * [Type1](#type1)
   * [Type2](#type2) 
 * [Audio Examples](#audio-examples)

## Codes for PHONEix

We have merged Muskits with ESPnet and the details of implementation can be found under https://github.com/A-Quarter-Mile/espnet/tree/alf.

### Installation

- The installation of ESPnet can follow the official doc [here](https://espnet.github.io/espnet/installation.html).

- Then, some extra packages for SVS need to be Installed. The installation scripts is in [tools/installers/install_muskit.sh](https://github.com/A-Quarter-Mile/espnet/blob/alf/tools/installers/install_muskit.sh).

### How to run

1. Download datasets

- The Japanese singing dataset Ofuton should be downloaded [here](https://sites.google.com/view/oftn-utagoedb/%E3%83%9B%E3%83%BC%E3%83%A0).
- The Chinese singing dataset Opencpop should be download [here](https://wenet.org.cn/opencpop/).

2. Change directory to the base directory
    ```bash
    # e.g.
    cd egs2/ofuton/svs1/
    ```
    You can perform any other recipes as the same way. e.g. `opencpop`.
    Keep in mind that all scripts should be ran at the level of `egs2/*/svs1`.

3. Change the configuration
    Describing the directory structure as follows:
    
    ```
    egs2/ofuton/svs1/
     - conf/      # Configuration files for training, inference, etc.
     - scripts/   # Bash utilities of muskit
     - pyscripts/ # Python utilities of muskit
     - utils/     # From Kaldi utilities
     - db.sh      # The directory path of each corpora
     - path.sh    # Setup script for environment variables
     - cmd.sh     # Configuration for your backend of job scheduler
     - run.sh     # Entry point
     - svs.sh     # Invoked by run.sh
    ```

    - You need to modify `db.sh` for specifying your corpus before executing `run.sh`. For example, when you touch the recipe of `egs2/ofuton`, you need to change the paths of `OFUTON`in `db.sh`.
    - `path.sh` is used to set up the environment for `run.sh`. Note that the Python interpreter used for ESPnet is not the current Python of your terminal, but it's the Python which was installed at `tools/`. Thus you need to source `path.sh` to use this Python.
        ```bash
        . path.sh
        python
        ```
    - `cmd.sh` is used for specifying the backend of the job scheduler. If you don't have such a system in your local machine environment, you don't need to change anything about this file. See [Using Job scheduling system](./parallelization.md)
    - `run.sh` is an example script to run all stages. To run the model with PHONEix, you need to set the `--train_config train_naive_rnn_dp_alf.yaml` for [LSTM-based model](https://arxiv.org/abs/2010.12024) and `--train_config train_xiaoice_alf.yaml` for [Transformer-based model](https://arxiv.org/pdf/2006.06261).
    
4. Run `run.sh`

    ```bash
    ./run.sh
    ```

    The procedures in `run.sh` can be divided into some stages, e.g. data preparation, training, and evaluation. You can specify the starting stage and the stopping stage.

    ```sh
    ./run.sh --stage 2 --stop-stage 6

    ```
    There are also some altenative otpions to skip specified stages:

    ```sh
    run.sh --skip_data_prep true  # Skip data preparation stages.
    run.sh --skip_train true      # Skip training stages.
    run.sh --skip_eval true       # Skip decoding and evaluation stages.
    run.sh --skip_upload false    # Enable packing and uploading stages.
    ```
    
### Evaluation

  We provide three objective evaluation metrics:

- Mel-cepstral distortion (MCD)
- Voiced / unvoiced error rate (VUV_E)
- Logarithmic rooted mean square error of the fundamental frequency (F![1](http://latex.codecogs.com/svg.latex?_0)RMSE). 

We apply dynamic time-warping (DTW) to match the length difference between ground-truth singing and generated singing.

Here we show the example command to calculate objective metrics:

```sh
cd egs2/<recipe_name>/svs1
. ./path.sh
# Evaluate MCD
./pyscripts/utils/evaluate_mcd.py \
    exp/<model_dir_name>/<decode_dir_name>/eval/wav/wav.scp \
    dump/raw/eval/wav.scp
```
While these objective metrics can estimate the quality of synthesized singing, it is still difficult to fully determine human perceptual quality from these values, especially with high-fidelity generated singing.
Therefore, we recommend performing the subjective evaluation (eg. MOS) if you want to check perceptual quality in detail.

You can refer [this page](https://github.com/kan-bayashi/webMUSHRA/blob/master/HOW_TO_SETUP.md) to launch web-based subjective evaluation system with [webMUSHRA](https://github.com/audiolabs/webMUSHRA).

## Codes for PHONEix

### Type1

We follow the processing pipelines in [Xiaoice](https://arxiv.org/pdf/2006.06261). It accepts score features at note level and expands frame lengths according to the phoneme duration produced by HMM-based forced-alignment.

The codes is in the same repositories as PHONEix, under https://github.com/A-Quarter-Mile/espnet/tree/alf. To run the model with Type1 strategy, you need to set the `--train_config train_naive_rnn_dp.yaml` for [LSTM-based model](https://arxiv.org/abs/2010.12024) and `--train_config train_xiaoice.yaml` for [Transformer-based model](https://arxiv.org/pdf/2006.06261). The other operations are the same as [Codes for PHONEix](#codes-for-phoneix)

### Type2

Type2 use the annotated phoneme time sequence as acoustic encoder input and length expansion ground truth for the length regulator. We analyze the acoustic feature follow [this work](https://arxiv.org/abs/2010.12024).

Codes can be found in [Muskit](https://github.com/A-Quarter-Mile/Muskits). Muskit follows ESPnet and Kaldi style data processing. The main structure and base codes are adapted from ESPnet. The [Installation](https://github.com/SJTMusicTeam/Muskits/wiki/Installation-Instructions) and [running instructions](https://github.com/SJTMusicTeam/Muskits/blob/main/doc/tutorial.md) are provided in Muskit.

To run the model with Type2 strategy in Muskit, you need to set the `--train_config train_naive_rnn_dp.yaml` for [LSTM-based model](https://arxiv.org/abs/2010.12024) and `--train_config train_xiaoice.yaml` for [Transformer-based model](https://arxiv.org/pdf/2006.06261). The other operations are the same as [Codes for PHONEix](#codes-for-phoneix)

## Audio Examples

It shows the comparison of the proposed acoustic processing strategy, PHONEix, with baselines (i.e., Type 1 and Type 2). It is evaluated with Bi-LSTM or Transformer based encoder-decoder structures on Ofuton and Opencpop.

- [Audio examples in Ofuton](https://github.com/A-Quarter-Mile/PHONEix/tree/main/Examples/Ofuton)
- [Audio examples in Opencpop](https://github.com/A-Quarter-Mile/PHONEix/tree/main/Examples/Opencpop)

```sh
Examples/dataset/model_strategy/*.wav
# e.g. Examples/Ofuton/LSTM_PHONEix/1.wav is the result for LSTM-based model with PHONEix strategy in Ofuton dataset.
```
