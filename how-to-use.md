# How to use

This is the tutorial how to run

## Setup

### Env Setup

If you are using other than Ubuntu, I suggest to use Docker. Here it's the tutorial:

1. Run docker container (`docker run ..`) with Ubuntu as the image. Here it's the example with running latest image of Ubuntu:
   ```bash
   docker run -itd --name sgnet-demo --net host -v <folder-in-host>:/<folder-in-container> ubuntu
   ```
   Note: Main directory of Ubuntu image is in `/` and main `~` path is `/root` and there is no sudo.

If you are already using Ubuntu, then you should be fine without using Docker, but you should have insert `sudo` prior to `apt` command

### Dependencies Setup

1. Clone repos
```bash
git clone --recurse-submodules -b runnable https://github.com/itsme-ranger/SGNet.pytorch.git
cd SGNet.pytorch
git clone --recurse-submodules -b runnable https://github.com/itsme-ranger/Trajectron-plus-plus.git
```
2. Download the SGNet weights [here](https://drive.google.com/drive/folders/1FCudihx-dmns-lh61uOcOD5uIWaKdKh8)
3. Install dependencies:
   1. `apt update`
   2. Install conda or miniconda
   3. Run:
      ```bash
      apt install -y build-essential  
      conda env create --file SGNet_env.yml
      conda activate SGNet
      python3 -m pip install --upgrade pip setuptools wheel
      cd Trajectron-plus-plus
      conda install --yes --file requirements-conda.txt
      python3 -m pip install -r requirements.txt
      ```
      **Important note:** you're might be encountered an error during this step. Please check [Solve for Setup Failure](#solve-for-setup-failure) section below

## Run predictors

1. Process ETH data
   ```bash
   cd Trajectron-plus-plus/experiments/pedestrians
   python process_data.py # This will take around 10-15 minutes, depending on your computer.
   cd ../../.. # to SGNet.pytorch
   mkdir -p data/ETHUCY
   cd data/ETHUCY
   ln -s Trajectron-plus-plus/experiments/pedestrians/raw/eth
   ```
2. To get evaluation metrics values, run SGNet predictors:
   1. back again to `SGNet.pytorch` folder
   2. For stochastic predictor: `python tools/ethucy/eval_cvae.py --gpu $CUDA_VISIBLE_DEVICES --dataset ETH --model SGNet_CVAE --checkpoint path/to/checkpoint`
   3. For deterministic predictor: `python tools/ethucy/eval_deterministic.py --gpu $CUDA_VISIBLE_DEVICES --dataset ETH --model SGNet --checkpoint path/to/checkpoint`
   4. If you run on CPU only or not using NVIDIA GPU, I suggest to omit `--gpu $CUDA_VISIBLE_DEVICES` for procedures 2.2 and 2.3
3. To get visualization
   1. Download [NuScenes mini](https://www.nuscenes.org/nuscenes#download) (You have to register first)
   2. Extract the contents into `SGNet.pytorch/Trajectron-plus-plus/experiments/nuScenes/v1.0-mini`
   3. Download `nuScenes-map-expansion-v1.2` and extract content of `maps` folder into `v1.0-mini/maps`
   4. Run this
      ```bash
      cd trajectron
      python test_online.py --log_dir=../experiments/nuScenes/models --data_dir=../experiments/processed --conf=config.json --eval_data_dict=nuScenes_test_mini_full.pkl --map_encoding --incl_robot_node
      ```
   5. For any other visualization option, follow [this docs](https://github.com/itsme-ranger/Trajectron-plus-plus/tree/runnable?tab=readme-ov-file#online-execution). Change line 110 of `trajectron/test_online.py` from `robot` into any other option based on table
   6. Check the simulation graphs/visualizations on `SGNet.pytorch/Trajectron-plus-plus/experiments/nuScenes/models/<option-choosed>/pred_figs`

## Solve for Setup Failure

Ideally, all those processes should be executed fine. However, there is a chance that the installation (2.2) will encounter an error. The anticipated error is failure of nuscene-devkit installation due to opencv-python installation error.

To solve this, you have to
1. navigate to any directory (no specific requirement here, but I suggest to navigate outer folder than `SGNet.Pytorch`). Then
2. Run
```bash
git clone -b 1.0.9  https://github.com/nutonomy/nuscenes-devkit.git
cd nuscenes-devkit
```
3. Open `setup/requirements.txt`, change `opencv-python` into `opencv-python-headless==4.1.1.26`
4. `export PYTHONPATH="${PYTHONPATH}:<YOUR-NUSCENE-DEVKIT-PARENT-PATH>/nuscenes-devkit/python-sdk"`
5. `python3 -m pip install -r setup/requirements.txt`
6. 