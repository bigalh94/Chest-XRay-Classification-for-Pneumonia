## Introduction
### The challenge
The rise in medical imaging has had the benefit of providing early detection of desease leading to early intervention. It has also resulted in reduced use of invasive procedures. And with increase in data the burden in medical experts examining that data increases.
<p></p>
According to this [article](https://www.gehealthcare.com/article/beyond-imagingthe-paradox-of-ai-and-medical-imaging-innovation) hospitals are producing 50 petabytes of data per yea resultin in 90% of all healthcare data. And this [article](https://missinglink.ai/guides/deep-learning-healthcare/what-you-need-to-know-about-deep-learning-medical-imaging/) states - 'The number of medical images that emergency room radiologists have to analyze can be overwhelming, with each medical study involving up to 3,000 images taking up 250GB of data.'
<p></p>
Therefore, we are in an age where there has been rapid growth in medical image acquisition as well as running challenging and interesting analysis on them. In addition, the overwhelming load of data and the need for rapid analysis is a recipe for error.

### Proposed solution
This is where technologies such as machine learning and deep learning can help to reduce the burden on the technicians and doctors. These technologies can also be beneficial if it is demonstrated that they are able to find patterns in images that are not easily detectable through human inspection. 

We propose to explore the employment of `Machine Learning` and `Deep Learning` algorithms to aid in the analysis of medical images. Specifically, this effort will focus on the analysis of chest X-ray images of patients to determine whether or not they had pneumonia.

The approach that we take is to build several models of different types and tuning which are then compared to determine the best performing model. The models are built using Convolutonal Neural Networks (CNN) based on the Keras / Tensorflow framework.

### Model evaluation
The models are evaluted on how acurrately they predict desease. In this case it is desired to have a high rate of `True Positives` and low rate of `False Negative`. This is because higher the `False Negative` the greater the number of deseased cases that are missed. It is preffered to misdiagnose as having desease, which would lead to furthur analysis, than missing the desease. The later could lead to a loss of valuable time in treatment.

Therefore, the metric that we use needs to consider the high cost associated with `False Negative`. Here is a review of the options.

#### Precision
$\Large precision = \frac{TP}{TP + FP}$

The focus of precision is the cost of high `False Positive` results. It measures the cost of patients being incorrectly diagnosed as having the desease. While this is a problem, it is not as significant as being misdiagnose as not having the desease.

### Recall
$\Large recall = \frac{TP}{TP + FN}$

Recall focuses on the cost of `False Negative` results. This would seem to be the metric that our models should be evaluated on. By reducing the number of `False Negative`, the model will reduce the number of deseased patients that are missed in diagnosis.

### F1 Score
$\Large f1 score = 2 x \frac{precision * recall}{precisoin + recall}$

This measure strikes a balance between `precision` and `recall`. While this seems to be a good compromise, you have to consider the the compromise might come at the cost of a higher recall than acceptable.

### Accuracy
$\Large accuracy = \frac{TP+TN}{TP+TN+FP+FN}$

This metric carries a higher probability of misclassification.

### Conclusion on evaluation metric
Base on the above, it is clear that recall is the metric that best addresses the problem. It gives a higher cost to `False Negative` results, meaning that less patients with desease will be misdiagnosed.## The Data
The dataset comes from Kermany et al. on [Mendeley](https://data.mendeley.com/datasets/rscbjbr9sj/3). There is also a version on [Kaggle](https://www.kaggle.com/paultimothymooney/chest-xray-pneumonia) which is a subset of the Mendeley dataset. However, the code for downloading the dataset is included in the Jupyter Notebook titled `01-Project_setup.ipynb`.

## The Data
The dataset comes from Kermany et al. on [Mendeley](https://data.mendeley.com/datasets/rscbjbr9sj/3). There is also a version on [Kaggle](https://www.kaggle.com/paultimothymooney/chest-xray-pneumonia) which is a subset of the Mendeley dataset. However, the code for downloading the dataset is included in the Jupyter Notebook titled `01-Project_setup.ipynb`.

## Project Setup
The notebook titled `*01-Project_setup.ipynb*` contains code that does the following:
1. Creates the `artifacts` directory tree structure.
2. Creates the `data` directory tree structure.
3. Downloads the zipped data from Mendeley.com
4. Creates the `train`, `valid`, and `test` folders 
5. Unzips the zipped files into the proper folder.
6. Creates a folder called preprocessed to hold augmented data
7. Runs augmentation on the `train` dataset which is heavily imbalanced on the side of deseased.
8. Copies files from `train` into the preprocessed folder to create a larger dataset.
9. Processes the data in preprocossed including flattening all of the image arrays so that they can be process by XGBoost.
10. Save the flatten images into numpy array to be used later.

---
---
## Keras CNN from scratch
In this experiment we create two Keras / Tensorflow CNN models. They are very basic and only differ in the number of layers.
> The code for this experiment can be found in the notebook titled `03-Basic CNN model.ipynb`
### Basic 
#### Data generation
We use an Keras ImageDataGenerator to augment the data. The code can be found in the notebooke titled `*03-Basic CNN model.ipynb*`

```Python
def batch_make(path, classes, batch_size=10, shuffle=True):
  batches = ImageDataGenerator(
      rescale=1./255,
        rotation_range=10,
        samplewise_center=True,
        samplewise_std_normalization=True,
        width_shift_range=0.2,
        height_shift_range=0.2,
        shear_range=0.2,
        zoom_range=0.2,
        fill_mode="nearest",
        cval=0.0,
        horizontal_flip=True).flow_from_directory(
            directory=path,
            target_size=(224,224),
            classes=classes,
            batch_size=batch_size,
            shuffle=shuffle
            )
  return batches


train_batch = batch_make(train_path, ['NORMAL', 'PNEUMONIA'], batch_size=20)
valid_batch = batch_make(valid_path, ['NORMAL', 'PNEUMONIA'], batch_size=20)
test_batch = batch_make(test_path, ['NORMAL', 'PNEUMONIA'], batch_size=20, shuffle=False)
```
#### Model creation
We created a simple convolutional neural network consisting of two convolutional layers with max pooling, and an output layer with two nodes. Below is the configuration.

```python
model = Sequential([
    Conv2D(filters=32, kernel_size=(3, 3), activation='relu', padding = 'same', input_shape=(224,224,3)),
    MaxPool2D(pool_size=(2, 2), strides=2),
    Conv2D(filters=64, kernel_size=(3, 3), activation='relu', padding = 'same'),
    MaxPool2D(pool_size=(2, 2), strides=2),
    Flatten(),
    Dense(units=2, activation='softmax')
])

```
#### Train the model
The model is trained for 100 epochs.
```python
history_1 = model.fit(x=train_batch,
                      steps_per_epoch=len(train_batch),
                      validation_data=valid_batch,
                      validation_steps=len(valid_batch),
                      epochs=100,
                      verbose=2
                      )

Epoch 1/100
210/210 - 1532s - loss: 0.3867 - accuracy: 0.8232 - val_loss: 0.2814 - val_accuracy: 0.8701
Epoch 2/100
210/210 - 104s - loss: 0.2613 - accuracy: 0.8908 - val_loss: 0.2403 - val_accuracy: 0.9064
Epoch 3/100
210/210 - 104s - loss: 0.2472 - accuracy: 0.8906 - val_loss: 0.2281 - val_accuracy: 0.9007
Epoch 4/100
210/210 - 104s - loss: 0.2427 - accuracy: 0.8956 - val_loss: 0.2286 - val_accuracy: 0.9179
Epoch 5/100
210/210 - 103s - loss: 0.2357 - accuracy: 0.9059 - val_loss: 0.2385 - val_accuracy: 0.9121
Epoch 6/100
210/210 - 103s - loss: 0.2299 - accuracy: 0.9023 - val_loss: 0.2172 - val_accuracy: 0.9188
Epoch 7/100
210/210 - 102s - loss: 0.2078 - accuracy: 0.9147 - val_loss: 0.2030 - val_accuracy: 0.9312
Epoch 8/100
210/210 - 102s - loss: 0.2167 - accuracy: 0.9097 - val_loss: 0.2141 - val_accuracy: 0.9160
Epoch 9/100
210/210 - 102s - loss: 0.2100 - accuracy: 0.9176 - val_loss: 0.2128 - val_accuracy: 0.9188
Epoch 10/100
210/210 - 102s - loss: 0.2064 - accuracy: 0.9109 - val_loss: 0.2227 - val_accuracy: 0.9074
Epoch 11/100
210/210 - 102s - loss: 0.2065 - accuracy: 0.9183 - val_loss: 0.1775 - val_accuracy: 0.9245
Epoch 12/100
210/210 - 102s - loss: 0.2050 - accuracy: 0.9173 - val_loss: 0.1916 - val_accuracy: 0.9255
Epoch 13/100
210/210 - 101s - loss: 0.2003 - accuracy: 0.9188 - val_loss: 0.2402 - val_accuracy: 0.8997
Epoch 14/100
210/210 - 101s - loss: 0.2023 - accuracy: 0.9221 - val_loss: 0.2370 - val_accuracy: 0.9045
Epoch 15/100
210/210 - 101s - loss: 0.1929 - accuracy: 0.9214 - val_loss: 0.1757 - val_accuracy: 0.9293
Epoch 16/100
210/210 - 101s - loss: 0.1786 - accuracy: 0.9252 - val_loss: 0.1970 - val_accuracy: 0.9274
Epoch 17/100
210/210 - 100s - loss: 0.1828 - accuracy: 0.9324 - val_loss: 0.1841 - val_accuracy: 0.9322
Epoch 18/100
210/210 - 99s - loss: 0.1835 - accuracy: 0.9274 - val_loss: 0.1825 - val_accuracy: 0.9226
Epoch 19/100
210/210 - 100s - loss: 0.1847 - accuracy: 0.9240 - val_loss: 0.1734 - val_accuracy: 0.9360
Epoch 20/100
210/210 - 100s - loss: 0.1768 - accuracy: 0.9295 - val_loss: 0.1657 - val_accuracy: 0.9427
Epoch 21/100
210/210 - 100s - loss: 0.1796 - accuracy: 0.9274 - val_loss: 0.1582 - val_accuracy: 0.9456
Epoch 22/100
210/210 - 100s - loss: 0.1750 - accuracy: 0.9295 - val_loss: 0.1644 - val_accuracy: 0.9322
Epoch 23/100
210/210 - 100s - loss: 0.1724 - accuracy: 0.9333 - val_loss: 0.1697 - val_accuracy: 0.9360
Epoch 24/100
210/210 - 100s - loss: 0.1745 - accuracy: 0.9338 - val_loss: 0.1778 - val_accuracy: 0.9255
Epoch 25/100
210/210 - 101s - loss: 0.1739 - accuracy: 0.9333 - val_loss: 0.1881 - val_accuracy: 0.9331
Epoch 26/100
210/210 - 100s - loss: 0.1867 - accuracy: 0.9278 - val_loss: 0.1671 - val_accuracy: 0.9351
Epoch 27/100
210/210 - 100s - loss: 0.1670 - accuracy: 0.9329 - val_loss: 0.1864 - val_accuracy: 0.9265
Epoch 28/100
210/210 - 101s - loss: 0.1580 - accuracy: 0.9424 - val_loss: 0.1672 - val_accuracy: 0.9322
Epoch 29/100
210/210 - 101s - loss: 0.1744 - accuracy: 0.9309 - val_loss: 0.1779 - val_accuracy: 0.9236
Epoch 30/100
210/210 - 102s - loss: 0.1701 - accuracy: 0.9329 - val_loss: 0.1927 - val_accuracy: 0.9303
Epoch 31/100
210/210 - 102s - loss: 0.1585 - accuracy: 0.9391 - val_loss: 0.1315 - val_accuracy: 0.9542
Epoch 32/100
210/210 - 102s - loss: 0.1691 - accuracy: 0.9321 - val_loss: 0.1557 - val_accuracy: 0.9360
Epoch 33/100
210/210 - 102s - loss: 0.1571 - accuracy: 0.9395 - val_loss: 0.1602 - val_accuracy: 0.9408
Epoch 34/100
210/210 - 102s - loss: 0.1604 - accuracy: 0.9398 - val_loss: 0.1562 - val_accuracy: 0.9379
Epoch 35/100
210/210 - 101s - loss: 0.1628 - accuracy: 0.9326 - val_loss: 0.1901 - val_accuracy: 0.9293
Epoch 36/100
210/210 - 100s - loss: 0.1720 - accuracy: 0.9297 - val_loss: 0.1542 - val_accuracy: 0.9398
Epoch 37/100
210/210 - 101s - loss: 0.1497 - accuracy: 0.9427 - val_loss: 0.1774 - val_accuracy: 0.9303
Epoch 38/100
210/210 - 100s - loss: 0.1590 - accuracy: 0.9395 - val_loss: 0.1579 - val_accuracy: 0.9370
Epoch 39/100
210/210 - 100s - loss: 0.1590 - accuracy: 0.9362 - val_loss: 0.1788 - val_accuracy: 0.9198
Epoch 40/100
210/210 - 100s - loss: 0.1599 - accuracy: 0.9374 - val_loss: 0.1540 - val_accuracy: 0.9408
Epoch 41/100
210/210 - 100s - loss: 0.1455 - accuracy: 0.9436 - val_loss: 0.1481 - val_accuracy: 0.9475
Epoch 42/100
210/210 - 100s - loss: 0.1637 - accuracy: 0.9362 - val_loss: 0.1530 - val_accuracy: 0.9379
Epoch 43/100
210/210 - 101s - loss: 0.1568 - accuracy: 0.9403 - val_loss: 0.1642 - val_accuracy: 0.9331
Epoch 44/100
210/210 - 100s - loss: 0.1623 - accuracy: 0.9364 - val_loss: 0.1315 - val_accuracy: 0.9475
Epoch 45/100
210/210 - 100s - loss: 0.1496 - accuracy: 0.9448 - val_loss: 0.1723 - val_accuracy: 0.9312
Epoch 46/100
210/210 - 100s - loss: 0.1465 - accuracy: 0.9453 - val_loss: 0.1474 - val_accuracy: 0.9436
Epoch 47/100
210/210 - 100s - loss: 0.1463 - accuracy: 0.9446 - val_loss: 0.1367 - val_accuracy: 0.9446
Epoch 48/100
210/210 - 99s - loss: 0.1498 - accuracy: 0.9412 - val_loss: 0.1478 - val_accuracy: 0.9513
Epoch 49/100
210/210 - 101s - loss: 0.1428 - accuracy: 0.9427 - val_loss: 0.1550 - val_accuracy: 0.9398
Epoch 50/100
210/210 - 101s - loss: 0.1480 - accuracy: 0.9427 - val_loss: 0.1270 - val_accuracy: 0.9532
Epoch 51/100
210/210 - 100s - loss: 0.1475 - accuracy: 0.9424 - val_loss: 0.1395 - val_accuracy: 0.9427
Epoch 52/100
210/210 - 100s - loss: 0.1432 - accuracy: 0.9455 - val_loss: 0.1279 - val_accuracy: 0.9522
Epoch 53/100
210/210 - 100s - loss: 0.1496 - accuracy: 0.9398 - val_loss: 0.1538 - val_accuracy: 0.9303
Epoch 54/100
210/210 - 99s - loss: 0.1476 - accuracy: 0.9427 - val_loss: 0.1351 - val_accuracy: 0.9494
Epoch 55/100
210/210 - 100s - loss: 0.1500 - accuracy: 0.9407 - val_loss: 0.1465 - val_accuracy: 0.9398
Epoch 56/100
210/210 - 100s - loss: 0.1364 - accuracy: 0.9491 - val_loss: 0.1400 - val_accuracy: 0.9456
Epoch 57/100
210/210 - 100s - loss: 0.1437 - accuracy: 0.9438 - val_loss: 0.1960 - val_accuracy: 0.9150
Epoch 58/100
210/210 - 100s - loss: 0.1408 - accuracy: 0.9470 - val_loss: 0.1351 - val_accuracy: 0.9513
Epoch 59/100
210/210 - 100s - loss: 0.1387 - accuracy: 0.9472 - val_loss: 0.1366 - val_accuracy: 0.9465
Epoch 60/100
210/210 - 100s - loss: 0.1497 - accuracy: 0.9424 - val_loss: 0.1417 - val_accuracy: 0.9484
Epoch 61/100
210/210 - 100s - loss: 0.1407 - accuracy: 0.9455 - val_loss: 0.1871 - val_accuracy: 0.9312
Epoch 62/100
210/210 - 100s - loss: 0.1439 - accuracy: 0.9446 - val_loss: 0.1365 - val_accuracy: 0.9398
Epoch 63/100
210/210 - 100s - loss: 0.1371 - accuracy: 0.9455 - val_loss: 0.1742 - val_accuracy: 0.9417
Epoch 64/100
210/210 - 100s - loss: 0.1322 - accuracy: 0.9534 - val_loss: 0.1302 - val_accuracy: 0.9465
Epoch 65/100
210/210 - 101s - loss: 0.1501 - accuracy: 0.9424 - val_loss: 0.1312 - val_accuracy: 0.9417
Epoch 66/100
210/210 - 100s - loss: 0.1320 - accuracy: 0.9455 - val_loss: 0.1514 - val_accuracy: 0.9408
Epoch 67/100
210/210 - 100s - loss: 0.1362 - accuracy: 0.9460 - val_loss: 0.1238 - val_accuracy: 0.9542
Epoch 68/100
210/210 - 101s - loss: 0.1362 - accuracy: 0.9465 - val_loss: 0.1525 - val_accuracy: 0.9427
Epoch 69/100
210/210 - 100s - loss: 0.1305 - accuracy: 0.9517 - val_loss: 0.1547 - val_accuracy: 0.9503
Epoch 70/100
210/210 - 100s - loss: 0.1353 - accuracy: 0.9496 - val_loss: 0.1165 - val_accuracy: 0.9484
Epoch 71/100
210/210 - 100s - loss: 0.1391 - accuracy: 0.9491 - val_loss: 0.1215 - val_accuracy: 0.9551
Epoch 72/100
210/210 - 100s - loss: 0.1410 - accuracy: 0.9481 - val_loss: 0.1923 - val_accuracy: 0.9322
Epoch 73/100
210/210 - 100s - loss: 0.1388 - accuracy: 0.9455 - val_loss: 0.1274 - val_accuracy: 0.9465
Epoch 74/100
210/210 - 101s - loss: 0.1327 - accuracy: 0.9520 - val_loss: 0.1341 - val_accuracy: 0.9494
Epoch 75/100
210/210 - 100s - loss: 0.1282 - accuracy: 0.9529 - val_loss: 0.1208 - val_accuracy: 0.9580
Epoch 76/100
210/210 - 100s - loss: 0.1262 - accuracy: 0.9505 - val_loss: 0.1445 - val_accuracy: 0.9427
Epoch 77/100
210/210 - 100s - loss: 0.1286 - accuracy: 0.9493 - val_loss: 0.1027 - val_accuracy: 0.9647
Epoch 78/100
210/210 - 100s - loss: 0.1325 - accuracy: 0.9498 - val_loss: 0.1518 - val_accuracy: 0.9331
Epoch 79/100
210/210 - 100s - loss: 0.1330 - accuracy: 0.9458 - val_loss: 0.1928 - val_accuracy: 0.9179
Epoch 80/100
210/210 - 101s - loss: 0.1252 - accuracy: 0.9544 - val_loss: 0.1319 - val_accuracy: 0.9503
Epoch 81/100
210/210 - 100s - loss: 0.1373 - accuracy: 0.9431 - val_loss: 0.1352 - val_accuracy: 0.9427
Epoch 82/100
210/210 - 100s - loss: 0.1285 - accuracy: 0.9501 - val_loss: 0.1304 - val_accuracy: 0.9408
Epoch 83/100
210/210 - 101s - loss: 0.1278 - accuracy: 0.9458 - val_loss: 0.1162 - val_accuracy: 0.9580
Epoch 84/100
210/210 - 100s - loss: 0.1276 - accuracy: 0.9522 - val_loss: 0.1205 - val_accuracy: 0.9542
Epoch 85/100
210/210 - 100s - loss: 0.1226 - accuracy: 0.9558 - val_loss: 0.1371 - val_accuracy: 0.9503
Epoch 86/100
210/210 - 101s - loss: 0.1288 - accuracy: 0.9503 - val_loss: 0.1106 - val_accuracy: 0.9561
Epoch 87/100
210/210 - 100s - loss: 0.1188 - accuracy: 0.9589 - val_loss: 0.1233 - val_accuracy: 0.9542
Epoch 88/100
210/210 - 100s - loss: 0.1169 - accuracy: 0.9599 - val_loss: 0.1168 - val_accuracy: 0.9494
Epoch 89/100
210/210 - 100s - loss: 0.1272 - accuracy: 0.9541 - val_loss: 0.1089 - val_accuracy: 0.9618
Epoch 90/100
210/210 - 100s - loss: 0.1211 - accuracy: 0.9544 - val_loss: 0.1298 - val_accuracy: 0.9446
Epoch 91/100
210/210 - 100s - loss: 0.1254 - accuracy: 0.9548 - val_loss: 0.1306 - val_accuracy: 0.9551
Epoch 92/100
210/210 - 100s - loss: 0.1269 - accuracy: 0.9520 - val_loss: 0.1154 - val_accuracy: 0.9551
Epoch 93/100
210/210 - 100s - loss: 0.1356 - accuracy: 0.9503 - val_loss: 0.1419 - val_accuracy: 0.9475
Epoch 94/100
210/210 - 100s - loss: 0.1183 - accuracy: 0.9544 - val_loss: 0.1217 - val_accuracy: 0.9551
Epoch 95/100
210/210 - 100s - loss: 0.1289 - accuracy: 0.9470 - val_loss: 0.1322 - val_accuracy: 0.9542
Epoch 96/100
210/210 - 101s - loss: 0.1211 - accuracy: 0.9546 - val_loss: 0.1266 - val_accuracy: 0.9494
Epoch 97/100
210/210 - 100s - loss: 0.1254 - accuracy: 0.9527 - val_loss: 0.1229 - val_accuracy: 0.9551
Epoch 98/100
210/210 - 100s - loss: 0.1166 - accuracy: 0.9553 - val_loss: 0.1201 - val_accuracy: 0.9475
Epoch 99/100
210/210 - 101s - loss: 0.1254 - accuracy: 0.9503 - val_loss: 0.1167 - val_accuracy: 0.9494
Epoch 100/100
210/210 - 100s - loss: 0.1119 - accuracy: 0.9584 - val_loss: 0.1157 - val_accuracy: 0.9570
```                

#### Training results
We can see that training and validation converge nicely. And we can see the same for the loss. It appears that the model has generalized well. Validation accuracy is very nearly at .096 for both datasets.
![](artifacts/charts/basic_cnn_model_av.png)

#### Predict on the test set
Now we'll see how the model does in making predictions on the test set.

```python
predictions = model.predict(x=test_batch, verbose=0)
```
#### Confusion Matrix
We plot a confusion matrix to get an idea of how well th model performed.
![](artifacts/charts/basic_cnn_model_cm.png)

#### Metrics
Using the confusion matrix we'll calculate the variuos metrics for the model. The prediction target is the pneumonia class. So, if pneumonia is predicted and is correct, then that's a `True Positive`. 

---
##### Precision
$\large precision = \frac{TP}{TP + FP} = \frac{378}{378 + 72}$

$\large precision = \normalsize 0.84$

---
##### Recall
$\large recall = \frac{TP}{TP+FN} = \frac{378}{378+12}$

$\large recall \approx \normalsize 0.97$

---
##### F1 score
$\large f1 score = 2 x \frac{precisionxrecall}{precision+recall} = 2 x \frac{0.84X0.97}{0.84+0.97}$

$\large f1 score \approx \normalsize 0.90$

___
##### Accuracy
$\large accuracy = \frac{TP+TN}{TP+TN+FP+FN} = \frac{378+162}{378+162+72+12}$

$\large accuracy \approx 0.87$

#### Analysis
While the `accuracy` of the model would suggest that this is a relatively weak model, remember that the most important metric for this business case is the `recall`. The model scored a very respectible `0.97` for `recall`, meaning that it correctly predicted pneumonia for `97%` of all the images labeled as deseased. That said, missing 3% of pneumonia cases is still problematic. 

### CNN with four convolutional layers
> The code for this experiment can be found in the notebook titled `03-Basic CNN model.ipynb`
#### Model creation
Next we recreate the experiment, but with additional layers.

```python

model_2 = Sequential([
    Conv2D(filters=32, kernel_size=(3, 3), activation='relu', padding = 'same', input_shape=(224,224,3)),
    MaxPool2D(pool_size=(2, 2), strides=2),
    Conv2D(filters=32, kernel_size=(3, 3), activation='relu', padding = 'same'),
    MaxPool2D(pool_size=(2, 2), strides=2),
    Conv2D(filters=32, kernel_size=(3, 3), activation='relu', padding = 'same'),
    MaxPool2D(pool_size=(2, 2), strides=2),
    Conv2D(filters=64, kernel_size=(3, 3), activation='relu', padding = 'same'),
    MaxPool2D(pool_size=(2, 2), strides=2),
    Flatten(),
    Dense(units=2, activation='softmax')
])

```

#### Train the model
We train this model for 50 epochs.
```python
history_2 = model_2.fit(x=train_batch,
                        steps_per_epoch=len(train_batch),
                        validation_data=valid_batch,
                        validation_steps=len(valid_batch),
                        epochs=50,
                        verbose=2
                        )

210/210 - 400s - loss: 0.1723 - accuracy: 0.9321 - val_loss: 0.1863 - val_accuracy: 0.9150
Epoch 22/50
210/210 - 399s - loss: 0.1844 - accuracy: 0.9300 - val_loss: 0.1924 - val_accuracy: 0.9274
Epoch 23/50
210/210 - 405s - loss: 0.1674 - accuracy: 0.9381 - val_loss: 0.1508 - val_accuracy: 0.9379
Epoch 24/50
210/210 - 408s - loss: 0.1625 - accuracy: 0.9357 - val_loss: 0.1789 - val_accuracy: 0.9284
Epoch 25/50
210/210 - 402s - loss: 0.1696 - accuracy: 0.9319 - val_loss: 0.1447 - val_accuracy: 0.9436
Epoch 26/50
210/210 - 423s - loss: 0.1597 - accuracy: 0.9379 - val_loss: 0.1542 - val_accuracy: 0.9360
Epoch 27/50
210/210 - 418s - loss: 0.1650 - accuracy: 0.9336 - val_loss: 0.1548 - val_accuracy: 0.9360
Epoch 28/50
210/210 - 424s - loss: 0.1553 - accuracy: 0.9357 - val_loss: 0.1717 - val_accuracy: 0.9322
Epoch 29/50
210/210 - 402s - loss: 0.1454 - accuracy: 0.9470 - val_loss: 0.1433 - val_accuracy: 0.9417
Epoch 30/50
210/210 - 397s - loss: 0.1558 - accuracy: 0.9415 - val_loss: 0.1686 - val_accuracy: 0.9265
Epoch 31/50
210/210 - 395s - loss: 0.1519 - accuracy: 0.9391 - val_loss: 0.1500 - val_accuracy: 0.9446
Epoch 32/50
210/210 - 403s - loss: 0.1591 - accuracy: 0.9367 - val_loss: 0.1482 - val_accuracy: 0.9351
Epoch 33/50
210/210 - 405s - loss: 0.1467 - accuracy: 0.9450 - val_loss: 0.1369 - val_accuracy: 0.9484
Epoch 34/50
210/210 - 394s - loss: 0.1505 - accuracy: 0.9393 - val_loss: 0.1404 - val_accuracy: 0.9475
Epoch 35/50
210/210 - 401s - loss: 0.1472 - accuracy: 0.9405 - val_loss: 0.1522 - val_accuracy: 0.9436
Epoch 36/50
210/210 - 409s - loss: 0.1414 - accuracy: 0.9446 - val_loss: 0.1319 - val_accuracy: 0.9484
Epoch 37/50
210/210 - 395s - loss: 0.1452 - accuracy: 0.9427 - val_loss: 0.1474 - val_accuracy: 0.9446
Epoch 38/50
210/210 - 400s - loss: 0.1546 - accuracy: 0.9448 - val_loss: 0.1397 - val_accuracy: 0.9379
Epoch 39/50
210/210 - 403s - loss: 0.1349 - accuracy: 0.9501 - val_loss: 0.1697 - val_accuracy: 0.9322
Epoch 40/50
210/210 - 407s - loss: 0.1436 - accuracy: 0.9431 - val_loss: 0.1192 - val_accuracy: 0.9542
Epoch 41/50
210/210 - 396s - loss: 0.1446 - accuracy: 0.9458 - val_loss: 0.1253 - val_accuracy: 0.9522
Epoch 42/50
210/210 - 398s - loss: 0.1439 - accuracy: 0.9470 - val_loss: 0.1365 - val_accuracy: 0.9408
Epoch 43/50
210/210 - 405s - loss: 0.1417 - accuracy: 0.9472 - val_loss: 0.1580 - val_accuracy: 0.9312
Epoch 44/50
210/210 - 407s - loss: 0.1431 - accuracy: 0.9465 - val_loss: 0.1259 - val_accuracy: 0.9532
Epoch 45/50
210/210 - 404s - loss: 0.1344 - accuracy: 0.9513 - val_loss: 0.1245 - val_accuracy: 0.9551
Epoch 46/50
210/210 - 398s - loss: 0.1303 - accuracy: 0.9489 - val_loss: 0.1177 - val_accuracy: 0.9666
Epoch 47/50
210/210 - 399s - loss: 0.1417 - accuracy: 0.9472 - val_loss: 0.1288 - val_accuracy: 0.9551
Epoch 48/50
210/210 - 397s - loss: 0.1302 - accuracy: 0.9493 - val_loss: 0.1269 - val_accuracy: 0.9542
Epoch 49/50
210/210 - 407s - loss: 0.1182 - accuracy: 0.9532 - val_loss: 0.1139 - val_accuracy: 0.9599
Epoch 50/50
210/210 - 395s - loss: 0.1223 - accuracy: 0.9503 - val_loss: 0.1183 - val_accuracy: 0.9542
```

#### Training results
The results are much as they were for the previous model. This model as the last generalizes very well.
![](artifacts/charts/bigger_cnn_model_av.png)

#### Predict on the test set
```
predictions_2 = model_2.predict(x=test_batch, verbose=0)
```

#### Confusion Matrix
![](artifacts/charts/bigger_cnn_model_cm.png)

#### The metrics

---
##### Precision
$\large precision = \frac{TP}{TP + FP} = \frac{378}{378 + 69}$

$\large precision = \normalsize 0.85$

---
##### Recall
$\large recall = \frac{TP}{TP+FN} = \frac{378}{378+12}$

$\large recall \approx \normalsize 0.97$

---
##### F1 score
$\large f1 score = 2 x \frac{precisionxrecall}{precision+recall} = 2 x \frac{0.85X0.97}{0.85+0.97}$

$\large f1 score \approx \normalsize 0.91$

---
##### Accuracy
$\large accuracy = \frac{TP+TN}{TP+TN+FP+FN} = \frac{378+165}{378+165+69+12}$

$\large accuracy \approx \normalsize 0.87$

#### Analysis
The scores for this model are very similar to the previous model. In fact, when we consider `recall` there is no difference. One would have to conclude that considering the added complexity, there is no reason to choose this model over the previous.

## Keras Pre-trained Models - VGG 16
The next two experiments leverage the famous VGG 16 model proposed by K. Simonyan and A. Zisserman from the University of Oxford in the paper “Very Deep Convolutional Networks for Large-Scale Image Recognition”. Rather than relying on solely on our data to train the model, we use this already train model and then add our data to refined the training for our purpose.
> The code for these experiments can be found in the notebook titled `04-Transfer Learning with VGG 16 model.ipynb`

#### Load VGG 16
Here we load the VGG 16 model and then print a summary.
```python
vgg16_model = tf.keras.applications.vgg16.VGG16()
vgg16_model.summary()

Downloading data from https://storage.googleapis.com/tensorflow/keras-applications/vgg16/vgg16_weights_tf_dim_ordering_tf_kernels.h5
553467904/553467096 [==============================] - 5s 0us/step
Model: "vgg16"
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
input_1 (InputLayer)         [(None, 224, 224, 3)]     0         
_________________________________________________________________
block1_conv1 (Conv2D)        (None, 224, 224, 64)      1792      
_________________________________________________________________
block1_conv2 (Conv2D)        (None, 224, 224, 64)      36928     
_________________________________________________________________
block1_pool (MaxPooling2D)   (None, 112, 112, 64)      0         
_________________________________________________________________
block2_conv1 (Conv2D)        (None, 112, 112, 128)     73856     
_________________________________________________________________
block2_conv2 (Conv2D)        (None, 112, 112, 128)     147584    
_________________________________________________________________
block2_pool (MaxPooling2D)   (None, 56, 56, 128)       0         
_________________________________________________________________
block3_conv1 (Conv2D)        (None, 56, 56, 256)       295168    
_________________________________________________________________
block3_conv2 (Conv2D)        (None, 56, 56, 256)       590080    
_________________________________________________________________
block3_conv3 (Conv2D)        (None, 56, 56, 256)       590080    
_________________________________________________________________
block3_pool (MaxPooling2D)   (None, 28, 28, 256)       0         
_________________________________________________________________
block4_conv1 (Conv2D)        (None, 28, 28, 512)       1180160   
_________________________________________________________________
block4_conv2 (Conv2D)        (None, 28, 28, 512)       2359808   
_________________________________________________________________
block4_conv3 (Conv2D)        (None, 28, 28, 512)       2359808   
_________________________________________________________________
block4_pool (MaxPooling2D)   (None, 14, 14, 512)       0         
_________________________________________________________________
block5_conv1 (Conv2D)        (None, 14, 14, 512)       2359808   
_________________________________________________________________
block5_conv2 (Conv2D)        (None, 14, 14, 512)       2359808   
_________________________________________________________________
block5_conv3 (Conv2D)        (None, 14, 14, 512)       2359808   
_________________________________________________________________
block5_pool (MaxPooling2D)   (None, 7, 7, 512)         0         
_________________________________________________________________
flatten (Flatten)            (None, 25088)             0         
_________________________________________________________________
fc1 (Dense)                  (None, 4096)              102764544 
_________________________________________________________________
fc2 (Dense)                  (None, 4096)              16781312  
_________________________________________________________________
predictions (Dense)          (None, 1000)              4097000   
=================================================================
Total params: 138,357,544
Trainable params: 138,357,544
Non-trainable params: 0

```
The summary shows that there are 138,257.544 trainable paremeters and 1000 different classes.

#### Create the sequential model
We create the Keras sequential model and then iterrate over VGG 16, adding each layer to the new model. We add all but the last model. That will be replace with our own output layer. Finally, we iterate over the layers of the sequential model and freeze all of the layers. That will allow us to retain all of the training inherited from the VGG 16 model.
```python
# Create a Keras sequential model
seq_vgg16_model = Sequential()

# Iterrrate over the vgg16 model and add layers to the new model. Exclude final layer.
for layer in vgg16_model.layers[:-1]:
    seq_vgg16_model.add(layer)

# Iterrate over sequential model and freeze all the layers.    
for layer in seq_vgg16_model.layers:
    layer.trainable = False

```

#### Now add the new output layer
Now we add the ouput layer. Recall the the original `VGG 16` had 1000 nodes in the final (output) layer. We are replacing that with a two node output layer since we have only two classes to consider. Note that we could also have use a single binary node. Let's also take a look at what the model looks like now. 
```python
seq_vgg16_model.add(Dense(units=2, activation='softmax'))
seq_vgg16_model.summary()    

Model: "sequential"
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
block1_conv1 (Conv2D)        (None, 224, 224, 64)      1792      
_________________________________________________________________
block1_conv2 (Conv2D)        (None, 224, 224, 64)      36928     
_________________________________________________________________
block1_pool (MaxPooling2D)   (None, 112, 112, 64)      0         
_________________________________________________________________
block2_conv1 (Conv2D)        (None, 112, 112, 128)     73856     
_________________________________________________________________
block2_conv2 (Conv2D)        (None, 112, 112, 128)     147584    
_________________________________________________________________
block2_pool (MaxPooling2D)   (None, 56, 56, 128)       0         
_________________________________________________________________
block3_conv1 (Conv2D)        (None, 56, 56, 256)       295168    
_________________________________________________________________
block3_conv2 (Conv2D)        (None, 56, 56, 256)       590080    
_________________________________________________________________
block3_conv3 (Conv2D)        (None, 56, 56, 256)       590080    
_________________________________________________________________
block3_pool (MaxPooling2D)   (None, 28, 28, 256)       0         
_________________________________________________________________
block4_conv1 (Conv2D)        (None, 28, 28, 512)       1180160   
_________________________________________________________________
block4_conv2 (Conv2D)        (None, 28, 28, 512)       2359808   
_________________________________________________________________
block4_conv3 (Conv2D)        (None, 28, 28, 512)       2359808   
_________________________________________________________________
block4_pool (MaxPooling2D)   (None, 14, 14, 512)       0         
_________________________________________________________________
block5_conv1 (Conv2D)        (None, 14, 14, 512)       2359808   
_________________________________________________________________
block5_conv2 (Conv2D)        (None, 14, 14, 512)       2359808   
_________________________________________________________________
block5_conv3 (Conv2D)        (None, 14, 14, 512)       2359808   
_________________________________________________________________
block5_pool (MaxPooling2D)   (None, 7, 7, 512)         0         
_________________________________________________________________
flatten (Flatten)            (None, 25088)             0         
_________________________________________________________________
fc1 (Dense)                  (None, 4096)              102764544 
_________________________________________________________________
fc2 (Dense)                  (None, 4096)              16781312  
_________________________________________________________________
dense (Dense)                (None, 2)                 8194      
=================================================================
Total params: 134,268,738
Trainable params: 8,194
Non-trainable params: 134,260,544
```

Note that we now have `8,194` trainable parameters, instead of the `134,268,738`. That's because when we made all of those layer untrainable, we froze `134,260,544` parameters.

#### Compile and Train the model
We will train the model for 10 epochs.

```python
seq_vgg16_model.compile(optimizer=Adam(learning_rate=0.0001), loss='categorical_crossentropy', metrics=['accuracy'])

mod_history = seq_vgg16_model.fit(x=train_batch,
          steps_per_epoch=len(train_batch),
          validation_data=valid_batch,
          validation_steps=len(valid_batch),
          epochs=10,
          verbose=2
)

Epoch 1/10
419/419 - 1970s - loss: 0.2405 - accuracy: 0.8973 - val_loss: 0.1568 - val_accuracy: 0.9417
Epoch 2/10
419/419 - 66s - loss: 0.1164 - accuracy: 0.9579 - val_loss: 0.1343 - val_accuracy: 0.9465
Epoch 3/10
419/419 - 67s - loss: 0.0948 - accuracy: 0.9661 - val_loss: 0.1304 - val_accuracy: 0.9484
Epoch 4/10
419/419 - 67s - loss: 0.0835 - accuracy: 0.9668 - val_loss: 0.1191 - val_accuracy: 0.9513
Epoch 5/10
419/419 - 67s - loss: 0.0759 - accuracy: 0.9744 - val_loss: 0.1172 - val_accuracy: 0.9561
Epoch 6/10
419/419 - 67s - loss: 0.0689 - accuracy: 0.9742 - val_loss: 0.1218 - val_accuracy: 0.9513
Epoch 7/10
419/419 - 66s - loss: 0.0650 - accuracy: 0.9775 - val_loss: 0.1105 - val_accuracy: 0.9532
Epoch 8/10
419/419 - 67s - loss: 0.0587 - accuracy: 0.9797 - val_loss: 0.1111 - val_accuracy: 0.9551
Epoch 9/10
419/419 - 68s - loss: 0.0546 - accuracy: 0.9816 - val_loss: 0.1079 - val_accuracy: 0.9570
Epoch 10/10
419/419 - 68s - loss: 0.0512 - accuracy: 0.9816 - val_loss: 0.1096 - val_accuracy: 0.9551
```
#### Training results
This model generalizes very well.
![](artifacts/charts/seq_vgg16_model_av.png)

#### Predict on test
```python
predictions = seq_vgg16_model.predict(x=test_batch, steps=len(test_batch), verbose=0)


```
#### Confusion Matrix
![](artifacts/charts/seq_vgg16_model_cm.png)

#### Metrics
---
##### Precision
$\large precision = \frac{TP}{TP+FP} = \frac{383}{383+71}$

$\large precision = \normalsize 0.84$

---
##### Recall
$\large recall = \frac{TP}{TP+FN} = \frac{383}{383+7}$

$\large recall = \normalsize 0.98$

---
##### F1 score
$\large f1 score = \normalsize 2 x \large\frac{precision x recall}{precision + recall} = \normalsize 2 x \large\frac{0.84 x 0.98}{0.84 + 0.98}$

$\normalsize f1 score \approx \normalsize 0.90$

---
##### Accuracy
$\large accuracy = \frac{TP+TN}{TP+TN+FP+FN} = \frac{383+163}{383+163+71+7}$

$\large accuracy \approx \normalsize 0.88$

#### Analysis
With a `recall` score of `0.98`, this model performs really well. Accuracy and precision suffer due to the number of `False Positive`. Of course, when lives are on the line, a `1.8%` rate of `False Negative` is still relatively high. Also, about `30%` of those without desease were classified as having it. 

### VGG 16 model 2
In this experiment we take the previous model and freeze two less layer and see if it makes a difference. The result of doing this is that there will now be a mush greater number of trainable parameters.

#### Create the model
This is virtually the same as the last model except for the second for-loop. Here the last two layers are used for training, where as in the previous model we only used the last.

```python
# Create a Keras sequential model
seq_vgg16_model_2 = Sequential()

# Iterrrate over the vgg16 model and add layers to the new model. Exclude final layer.
for layer in vgg16_model.layers[:-1]:
    seq_vgg16_model_2.add(layer)

# Freeze all but the last two layers
for layer in seq_vgg16_model_2.layers[:-2]:
    layer.trainable = False
    
```
#### Add the output layer
This is the esact same output layer as in the previous model
```python
seq_vgg16_model_2.add(Dense(units=2, activation='softmax'))
seq_vgg16_model_2.summary()    

Model: "sequential"
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
block1_conv1 (Conv2D)        (None, 224, 224, 64)      1792      
_________________________________________________________________
block1_conv2 (Conv2D)        (None, 224, 224, 64)      36928     
_________________________________________________________________
block1_pool (MaxPooling2D)   (None, 112, 112, 64)      0         
_________________________________________________________________
block2_conv1 (Conv2D)        (None, 112, 112, 128)     73856     
_________________________________________________________________
block2_conv2 (Conv2D)        (None, 112, 112, 128)     147584    
_________________________________________________________________
block2_pool (MaxPooling2D)   (None, 56, 56, 128)       0         
_________________________________________________________________
block3_conv1 (Conv2D)        (None, 56, 56, 256)       295168    
_________________________________________________________________
block3_conv2 (Conv2D)        (None, 56, 56, 256)       590080    
_________________________________________________________________
block3_conv3 (Conv2D)        (None, 56, 56, 256)       590080    
_________________________________________________________________
block3_pool (MaxPooling2D)   (None, 28, 28, 256)       0         
_________________________________________________________________
block4_conv1 (Conv2D)        (None, 28, 28, 512)       1180160   
_________________________________________________________________
block4_conv2 (Conv2D)        (None, 28, 28, 512)       2359808   
_________________________________________________________________
block4_conv3 (Conv2D)        (None, 28, 28, 512)       2359808   
_________________________________________________________________
block4_pool (MaxPooling2D)   (None, 14, 14, 512)       0         
_________________________________________________________________
block5_conv1 (Conv2D)        (None, 14, 14, 512)       2359808   
_________________________________________________________________
block5_conv2 (Conv2D)        (None, 14, 14, 512)       2359808   
_________________________________________________________________
block5_conv3 (Conv2D)        (None, 14, 14, 512)       2359808   
_________________________________________________________________
block5_pool (MaxPooling2D)   (None, 7, 7, 512)         0         
_________________________________________________________________
flatten (Flatten)            (None, 25088)             0         
_________________________________________________________________
fc1 (Dense)                  (None, 4096)              102764544 
_________________________________________________________________
fc2 (Dense)                  (None, 4096)              16781312  
_________________________________________________________________
dense (Dense)                (None, 2)                 8194      
=================================================================
Total params: 134,268,738
Trainable params: 119,554,050
Non-trainable params: 14,714,688
```
The result of our change is that now there are 119,554,050 trainable parameters. 

#### Compile and Train the model
We train for 10 epochs.
```python
seq_vgg16_model_2.compile(optimizer=Adam(learning_rate=0.0001), loss='categorical_crossentropy', metrics=['accuracy'])


mod_2_history = seq_vgg16_model_2.fit(x=train_batch,
          steps_per_epoch=len(train_batch),
          validation_data=valid_batch,
          validation_steps=len(valid_batch),
          epochs=10,
          verbose=2
)

Epoch 1/10
419/419 - 2343s - loss: 0.1728 - accuracy: 0.9589 - val_loss: 0.0613 - val_accuracy: 0.9733
Epoch 2/10
419/419 - 93s - loss: 0.0702 - accuracy: 0.9823 - val_loss: 1.4422 - val_accuracy: 0.9112
Epoch 3/10
419/419 - 93s - loss: 0.0968 - accuracy: 0.9859 - val_loss: 0.2311 - val_accuracy: 0.9513
Epoch 4/10
419/419 - 93s - loss: 0.0113 - accuracy: 0.9967 - val_loss: 0.1169 - val_accuracy: 0.9819
Epoch 5/10
419/419 - 93s - loss: 3.1420e-04 - accuracy: 1.0000 - val_loss: 0.1316 - val_accuracy: 0.9838
Epoch 6/10
419/419 - 93s - loss: 1.7572e-05 - accuracy: 1.0000 - val_loss: 0.1285 - val_accuracy: 0.9847
Epoch 7/10
419/419 - 93s - loss: 1.4997e-05 - accuracy: 1.0000 - val_loss: 0.1449 - val_accuracy: 0.9847
Epoch 8/10
419/419 - 93s - loss: 1.0747e-06 - accuracy: 1.0000 - val_loss: 0.1481 - val_accuracy: 0.9866
Epoch 9/10
419/419 - 93s - loss: 4.6145e-08 - accuracy: 1.0000 - val_loss: 0.1498 - val_accuracy: 0.9866
Epoch 10/10
419/419 - 93s - loss: 3.3356e-08 - accuracy: 1.0000 - val_loss: 0.1509 - val_accuracy: 0.9857
```
#### Training results
The models generalizes quite well, but it is evident that the validation set displays some significant loss.
![](artifacts/charts/seq_vgg16_model_2_av.png)
`
#### Predict on test
```python
predictions_2 = seq_vgg16_model_2.predict(x=test_batch, steps=len(test_batch), verbose=0)
```
#### Confusion Matrix
![](artifacts/charts/seq_vgg16_model_2_cm.png)

#### Metrics
---
##### Precision
$\large precision = \frac{TP}{TP+FP} = \frac{388}{388+71} $

$\large precision \approx \normalsize 0.85$

---
##### Recall
$\large recall = \frac{TP}{TP+TN} = \frac{388}{388+2}$

$\large recall \approx \normalsize 0.99$

---
##### F1 score
$\large f1 score = \normalsize 2 x \frac{precision x recall}{precision + recall} = \normalsize 2 x \frac{0.85 x 0.99}{0.85 + 0.99}$

$\large f1 score \approx \normalsize 0.91$

---
##### Accuracy
$\large accuracy = \frac{TP+TN}{TP+TN+FP+FN} = \frac{388 + 163}{388 + 163 + 71 + 2}$

$\large accuracy \approx \normalsize 0.88$

#### Analysis
This time the recall score is `0.99` (actually close to 0.995). This time the number of patients with pneumonia that are misclassified is lower than `.5%`. This is a lot closer to what might be considered acceptable. There are still around `30%` of people without desease that are classified as having the ailment.

## Overall Analysis

We used the Keras / Tensorflow framework to create four CNN models. The first two models were created from scratch and trained only on the data at hand. Then we used the VGG 16 model to create and additional two models. The advantage of the VGG 16 model is that it has already been trained on an very large dataset of 1000 different classes. To this pretrained model we add some additional training to include our images. We chose to use recall as the metric to evaluate the performance of the models because it takes into account the cost of the `False Negative` results.

### Summary of results

|Model | Recall | Precision | F1 | Accuracy|
|----------|-----------|---------|--------|-------------------|
|Basic CNN with two conv layers| 0.97 | 0.84 | 0.90 | 0.87 |
|Basic CNN with four conv layers | 0.97 | 0.85 | 0.91 | 0.87 |
|VGG 16 Model One | 0.98 | 0.84 | 0.90 | 0.88 |
|VGG 16 Model Two | 0.99 | 0.85 | 0.91 | 0.88 |

We can see that the two basic CNN models did fairly well - both scoring 0.97 on the recall score. A perfect score of 1.0 would mean that there are no `False Negative` predictions. A lower score means higher `FN` predictions. While 0.97 seems impressive, we must remember that any `FN` prediction results in a patient not getting the treatment that they need.

The VGG 16 Model One had all layers except for the last one frozen so as to leverage what the model had learned when originally trained on a massive dataset. This meant that there were 8,194 paramaters that are trainable. As is visible above, this model scored `0.98` on recall. This is an improvement, but probably still lower than what one should expect when lives ar potentially at stake.

VGG 16 Model Two scored `0.99` on recall. But this was actually closer to `0.995`. This model had two additional layer set to trainable. This meant that there were `119,554,050` trainable parameters. This means more flexibilty to learn from our dataset.

## Final verdict
VGG 16 Model Two clearly performs better than the other three models base on our prformance evaluation metric of recall. With a low score of `0.99` it comes quite close to the kind of numbers that one would like to see in production. I believe that with some fine tuning and refinement, this model can be improved. 

## Future work
They say that Deep Learning models are black boxes. That we just know they work, but have no view into what they are doing. This is less true today. I would like to furthur evaluate these models using `activation maps`. `Activation maps` give us insight into what features the model finds interesting.
