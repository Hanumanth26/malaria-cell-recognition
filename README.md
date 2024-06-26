# Deep Neural Network for Malaria Infected Cell Recognition

## AIM

To develop a deep neural network for Malaria infected cell recognition and to analyze the performance.

## Problem Statement and Dataset
Aims to create an algorithm that can detect malaria-infected cells in blood samples using deep learning techniques. The goal is to develop a model that can accurately identify infected cells and distinguish them from healthy ones. The performance of the model will be evaluated based on its accuracy, precision, recall, and F1 score. This problem statement is important because it can help improve the diagnosis of malaria and reduce the time and cost associated with manual diagnosis.

Malaria dataset of 27,558 cell images with an equal number of parasitized and uninfected cells. A level-set based algorithm was applied to detect and segment the red blood cells. The images were collected and annotated by medical professionals.Here we build a convolutional neural network model that is able to classify the cells.

![193736032-b5847f1f-f002-4edc-912a-eaf48444f1b0](https://github.com/etjabajasphin/malaria-cell-recognition/assets/119218812/3704a85a-de49-47ec-bce6-a6dabd2102de)


## DESIGN STEPS

### STEP 1:
Import tensorflow and preprocessing libraries
### STEP 2:
Download and load the dataset folder
### STEP 3:
Split the training and testing folders.
### STEP 4:
Perform image data generation methods.
### STEP-5:
Build the convolutional neural network model
### STEP-6:
Train the model with the training data
### STEP-7:
Plot the performance plot
### STEP-8:
Evaluate the model with the testing data using probability prediction(uninfected-> prob>0.5,parasitized-> <=0.5)
### STEP-9:
Fit the model and predict the sample input.
## PROGRAM

### Name: hanumanth rao

### Register Number: 212222240016

```
# to share the GPU resources for multiple sessions
import tensorflow as tf
from tensorflow.compat.v1.keras.backend import set_session
config = tf.compat.v1.ConfigProto()
config.gpu_options.allow_growth = True # dynamically grow the memory used on the GPU
config.log_device_placement = True # to log device placement (on which device the operation ran)
sess = tf.compat.v1.Session(config=config)
set_session(sess)

import os
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
from matplotlib.image import imread
from tensorflow.keras.preprocessing.image import ImageDataGenerator
from tensorflow import keras
from tensorflow.keras import layers
from tensorflow.keras import utils
from tensorflow.keras import models
from sklearn.metrics import classification_report,confusion_matrix
import tensorflow as tf
%matplotlib inline

my_data_dir = './dataset/cell_images'
os.listdir(my_data_dir)
test_path = my_data_dir+'/test/'
train_path = my_data_dir+'/train/'
os.listdir(train_path)
len(os.listdir(train_path+'/uninfected/'))
len(os.listdir(train_path+'/parasitized/'))
os.listdir(train_path+'/parasitized')[0]
para_img= imread(train_path+
                 '/parasitized/'+
                 os.listdir(train_path+'/parasitized')[0])
plt.imshow(para_img)

# Checking the image dimensions
dim1 = []
dim2 = []
for image_filename in os.listdir(test_path+'/uninfected'):
    img = imread(test_path+'/uninfected'+'/'+image_filename)
    d1,d2,colors = img.shape
    dim1.append(d1)
    dim2.append(d2)
sns.jointplot(x=dim1,y=dim2)

image_shape = (130,130,3)
help(ImageDataGenerator)
image_gen = ImageDataGenerator(rotation_range=20, # rotate the image 20 degrees
                               width_shift_range=0.10, # Shift the pic width by a max of 5%
                               height_shift_range=0.10, # Shift the pic height by a max of 5%
                               rescale=1/255, # Rescale the image by normalzing it.
                               shear_range=0.1, # Shear means cutting away part of the image (max 10%)
                               zoom_range=0.1, # Zoom in by 10% max
                               horizontal_flip=True, # Allo horizontal flipping
                               fill_mode='nearest' # Fill in missing pixels with the nearest filled value
                              )
image_gen.flow_from_directory(train_path)
image_gen.flow_from_directory(test_path)
model = models.Sequential()
model.add(layers.Conv2D(filters=64, kernel_size=(3,3),input_shape=image_shape, activation='relu',))
model.add(layers.MaxPooling2D(pool_size=(2, 2)))

model.add(layers.Conv2D(filters=32, kernel_size=(5,5), activation='relu',))
model.add(layers.MaxPooling2D(pool_size=(2, 2)))

model.add(layers.Conv2D(filters=16, kernel_size=(3,3), activation='relu',))
model.add(layers.MaxPooling2D(pool_size=(2, 2)))


model.add(layers.Flatten())

model.add(layers.Dense(128))
model.add(layers.Activation('relu'))
model.add(layers.Dropout(0.5))


model.add(layers.Dense(64))
model.add(layers.Activation('relu'))

model.add(layers.Dense(1))
model.add(layers.Activation('sigmoid'))

model.compile(loss='binary_crossentropy',
              optimizer='adam',
              metrics=['accuracy'])
model.summary()

batch_size = 16
help(image_gen.flow_from_directory)

train_image_gen = image_gen.flow_from_directory(train_path,
                                               target_size=image_shape[:2],
                                                color_mode='rgb',
                                               batch_size=batch_size,
                                               class_mode='binary')

train_image_gen.batch_size
len(train_image_gen.classes)
train_image_gen.total_batches_seen

test_image_gen = image_gen.flow_from_directory(test_path,
                                               target_size=image_shape[:2],
                                               color_mode='rgb',
                                               batch_size=batch_size,
                                               class_mode='binary',shuffle=False)
train_image_gen.class_indices
results = model.fit(train_image_gen,epochs=3,validation_data=test_image_gen)

model.save('cell_model.h5')
losses = pd.DataFrame(model.history.history)
losses[['loss','val_loss']].plot()

model.metrics_names
model.evaluate(test_image_gen)
pred_probabilities = model.predict(test_image_gen)
test_image_gen.classes

predictions = pred_probabilities > 0.5
print(classification_report(test_image_gen.classes,predictions))
confusion_matrix(test_image_gen.classes,predictions)

```
## OUTPUT

### Training Loss, Validation Loss Vs Iteration Plot

![image](https://github.com/Sriram8452/malaria-cell-recognition/assets/118708032/390a25bf-9511-4438-9293-20947860bbe6)


### Classification Report

![image](https://github.com/Sriram8452/malaria-cell-recognition/assets/118708032/13dc5e37-c79b-48a1-abb0-274a0b172811)


### Confusion Matrix

![image](https://github.com/Sriram8452/malaria-cell-recognition/assets/118708032/178fc0ee-2021-4a8e-9d45-7926ae0ff3fb)



### New Sample Data Prediction

![image](https://github.com/Sriram8452/malaria-cell-recognition/assets/118708032/8ad11130-ecc6-4f4b-9e79-1b480470679e)



## RESULT:
Thus a deep neural network for Malaria infected cell recognition and to analyze the performance is created using tensorflow.

