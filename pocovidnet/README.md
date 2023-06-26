# pocovidnet
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![build](https://github.com/jannisborn/covid19_ultrasound/actions/workflows/build.yml/badge.svg)](https://github.com/jannisborn/covid19_ultrasound/actions/workflows/build.yml)

A simple package to train deep learning models on ultrasound data for COVID19.

## Train/test split

[//]: # (**Due to multiple papers that used our dataset incorrectly, we are adding the following disclaimer: Please make sure to create a meaningful train/test data split. Do not split the data on a frame-level, but on a video/patient-level. The task becomes trivial otherwise because consecutive LUS frames are extremely correlated. We provide scripts to create a cross-validation split for you. See the instructions [here]&#40;#cross-validation-splitting&#41;.**)

[//]: # ()
[//]: # (For training, we used data by the Peru team that includes 14 patients with both normal and abnormal classified ultrasound images.)

[//]: # (The clips from the ultrasound device are predominantly from the left and right posterier region of the patients. The images converted from the)

[//]: # (video consists of both curvilinear and linear frames.)

For training, we have utilised 501 abnormal and 452 normal frames. 
For test, we have utilised 62 patient clip with an average of 510 clips on an average. 

## Installation

The library itself has few dependencies (see [setup.py](setup.py)) with loose requirements. 

To run the code, just install the package `pocovidnet` in editable mode:

```sh
git clone https://github.com/BorgwardtLab/covid19_ultrasound.git
cd covid19_ultrasound/pocovidnet/
pip install -e .
```

## Training the model

For the database part, can use the POCOVIDNET dataset with two
class labels instead of 3 or 4.
For the required database, run the following instructions for database set up. 

### Set up database

Open and run the following notebook `scripts/dataset-builder.ipynb` to utilize the 
clips in image format datset

OR

A lot of data is directly provided in this repository in the [data folder](../data).

#### Web data
Parts of our database are videos/images from online sources that are not licensed for redistribution. This includes publications with restrictive licenses (e.g. from Elsevier) or data from commercial websites. These samples are not provided within our repo but we provide a script to download and preprocess this data automatically:
```sh
cd ../data
sh get_and_process_web_data.sh
```
This will take a while, but afterwards more data will be in the [data folder](../data).

### Videos to images

First, we have to merge the videos and images to create an image dataset. 
You can use the script `cross_val_splitter.py` to copy from [pocus images](../data/pocus_images) and [pocus videos](../data/pocus_videos). It will cope the images automatically and process all videos (read the frames and save every x-th frame dependent on the framerate supplied in args).

Note: In the script, it is hard-coded that only convex POCUS data is taken, and only the classes `covid`, `pneumonia`, `regular` (there is not enough data for `viral`yet). You can change this selection in the script.

From the directory of this README, execute:
```sh
python3 scripts/build_image_dataset.py
```

Now, your [data folder](../data) should contain a new folder `image_dataset`
with folders `covid`, `pneumonia`, `regular` and `viral` or a subset of those dependent on your selection.


### Cross validation splitting

#### Following step not required for LUS 2022-23 dataset

**For our train/test we have used one train/test split that includes the existing 12-14 clips and additional 62 clips. Hence, the following 
splits are not required.
   
The next step is to perform the data split. You can use the script
`cross_val_splitter.py` to perform a 5-fold cross validation (it will use the data from `data/image_dataset` by default):

From the directory of this README, execute:
```sh
python3 scripts/cross_val_splitter.py --splits 5
```
Now, your [data folder](../data) should contain a new folder `cross_validation`
with folders `fold_1`, `fold_2`. Each folder contains only the test data for
that specific fold.

#### Uninformative data
If you want to add data from an *uninformative* class, see [here](https://github.com/jannisborn/covid19_pocus_ultrasound/tree/master/data#add-class-uninformative).


### Train the model

Afterwards you can train the model by:
For test set 1 (interquartile frame range)
```sh
python3 scripts/train_covid19.py -d ../data/mid_cross_val/ -f 1 -ep 20 -m scripts/models-og
```
For test set 1 (Alternate frame range)
```sh
python3 scripts/train_covid19.py -d ../data/alt_cross_val/ -f 1 -ep 20 -m scripts/models-og-copy
```

*NOTE*: `train_covid19.py` will automatically utilize the data from first fold for training and other
for test.

### Clip-level classification

Run the notebook `scripts/frame-classifier.ipynb`