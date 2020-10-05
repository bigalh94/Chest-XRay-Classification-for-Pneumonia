# Chest XRay Classification for Pneumonia
<img src="https://github.com/bigalh94/Chest-XRay-Classification-for-Pneumonia/blob/master/sample_img/NORMAL-179015-0001.jpeg" width="200" hight="200"><img src= "https://github.com/bigalh94/Chest-XRay-Classification-for-Pneumonia/blob/master/sample_img/NORMAL-183773-0001.jpeg" width="200" height="200"><img src="https://github.com/bigalh94/Chest-XRay-Classification-for-Pneumonia/blob/master/sample_img/NORMAL-202916-0003.jpeg" width="200" height="200"><img src="https://github.com/bigalh94/Chest-XRay-Classification-for-Pneumonia/blob/master/sample_img/NORMAL-87870-0001.jpeg" width="200" height="200">
### Author: Alvaro Henriquez
## Introduction
Machine Learning and Deep Learning continue to play a an ever increasing roll in the medical field. Medical imaging is one of those area where these technologies can greatly contribute in increasing the acurracy of analysis, while decreasing some of the workload of technicians and doctors.<p></p>
In this proof of concept (POC) we build several different models and compare them to each other to determine which performs best base on metrics chosen for the specific problem. The models are based on the XGBoost algorithm and on convolutional neural networks (CNN) using the Keras/Tensorflow framework.

## The Data
The dataset comes from Kermany et al. on [Mendeley](https://data.mendeley.com/datasets/rscbjbr9sj/3). There is also a version on [Kaggle](https://www.kaggle.com/paultimothymooney/chest-xray-pneumonia) which is a subset of the Mendeley dataset. However, the code for downloading the dataset is included in the Jupyter Notebook titled `01-Project_setup.ipynb`.

## Requirements
This project was created using Google Colab. All of the required libraries are included in the Colab environment except for `Baysian-Optimization`. This is also taken care of by `01-Project_setup.ipynb`, which installs the library with `pip install bayesian-optimization` as the first line of code in the notebook. You will need a Google account in order to use Colab.
<p></p>


