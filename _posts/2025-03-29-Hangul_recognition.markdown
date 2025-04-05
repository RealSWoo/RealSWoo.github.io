---
layout: post
title: Hangul Recognition OCR
date: 2025-04-05 9:31:29 +0900
categories: Code
---

This is code

```ruby
import tensorflow as tf
from tensorflow import keras

# dataset and transformation
import io
import os
import cv2
import math
import glob
from sklearn.model_selection import train_test_split
from tensorflow.keras.preprocessing.image import ImageDataGenerator

os.environ["CUDA_VISIBLE_DEVICES"] = ""

# display images
import matplotlib.pyplot as plt 
%matplotlib inline

#!sudo apt-get install -y fonts-nanum
#!sudo fc-cache -fv
#!rm ~/.cache/matplotlib -rf

plt.rc('font', family='NanumBarunGothic')

# utils
import numpy as np
import pandas as pd
import time
import copy

!wget -O dataset.zip https://raw.githubusercontent.com/kitae0522/hangul-recognition/main/data/dataset.zip
!wget -O common-hangul-syllable.txt https://raw.githubusercontent.com/kitae0522/hangul-recognition/main/data/common-hangul-syllable.txt
!wget -O train.csv https://raw.githubusercontent.com/kitae0522/hangul-recognition/main/train.csv

!unzip -qq ./dataset.zip -d ./data

dataset = glob.glob(os.path.join('./data', '*.jpeg'))
print(len(dataset))

with io.open('./common-hangul-syllable.txt', 'r', encoding='utf-8') as f:
    syllable_list = f.read().splitlines()
    
# encode dict
s_to_idx = {syllable:idx for idx, syllable in enumerate(syllable_list)}
idx_to_s = {idx:syllable for idx, syllable in enumerate(syllable_list)}
num_class = syllable_list.__len__()

df = pd.read_csv('./train.csv')

print(num_class, df.__len__())
df.head(5)

xs = np.array(df['path'])
ys = np.array([s_to_idx[item] for item in df['label']])

train_x, valid_x, train_y, valid_y = train_test_split(xs, ys, test_size=0.2)
train_x, test_x, train_y, test_y = train_test_split(xs, ys, test_size=0.2)

train_x = np.array([cv2.imread(item, cv2.IMREAD_GRAYSCALE) for item in train_x]).reshape(train_x.shape[0], 48, 48, 1)
valid_x = np.array([cv2.imread(item, cv2.IMREAD_GRAYSCALE) for item in valid_x]).reshape(valid_x.shape[0], 48, 48, 1)
test_x = np.array([cv2.imread(item, cv2.IMREAD_GRAYSCALE) for item in test_x]).reshape(test_x.shape[0], 48, 48, 1)
print(train_x.shape, valid_x.shape, test_x.shape)
print(train_y.shape, valid_y.shape, test_y.shape)

def show_sample(image, label, sample_count=25):
    grid_count = math.ceil(math.ceil(math.sqrt(sample_count)))
    grid_cout = min(grid_count, len(image), len(label))
    plt.figure(figsize=(2*grid_count, 2*grid_count))

    for i in range(sample_count):
        plt.subplot(grid_count, grid_count, i+1)
        plt.xticks([])
        plt.yticks([])
        plt.grid(False)
        plt.imshow(image[i].reshape(48, 48), cmap=plt.cm.gray)
        plt.xlabel(idx_to_s[label[i]], fontsize=14)
    plt.show()
show_sample(train_x, train_y)

# 기존 vgg19 model에서 일부 layer을 수정했습니다.

X = keras.layers.Input(shape=[48, 48, 1])

# 1
H = keras.layers.Conv2D(filters=64, kernel_size=3, padding='same', activation='relu', name='conv2d_1_1')(X)
H = keras.layers.Conv2D(filters=64, kernel_size=3, padding='same', activation='relu', name='conv2d_1_2')(H)

# 2
H = keras.layers.MaxPooling2D(pool_size=(2, 2), padding='same', name='maxpool2D_1')(H)
H = keras.layers.Conv2D(filters=128, kernel_size=3, padding='same', activation='relu', name='conv2d_2_1')(H)
H = keras.layers.Conv2D(filters=128, kernel_size=3, padding='same', activation='relu', name='conv2d_2_2')(H)

H = keras.layers.BatchNormalization()(H)
H = keras.layers.ReLU()(H)

# 3
H = keras.layers.MaxPooling2D(pool_size=(2, 2), padding='same', name='maxpool2D_2')(H)
H = keras.layers.Conv2D(filters=256, kernel_size=3, padding='same', activation='relu', name='conv2d_3_1')(H)
H = keras.layers.Conv2D(filters=256, kernel_size=3, padding='same', activation='relu', name='conv2d_3_2')(H)
H = keras.layers.Conv2D(filters=256, kernel_size=3, padding='same', activation='relu', name='conv2d_3_3')(H)
H = keras.layers.Conv2D(filters=256, kernel_size=3, padding='same', activation='relu', name='conv2d_3_4')(H)

H = keras.layers.BatchNormalization()(H)
H = keras.layers.ReLU()(H)

# 4
H = keras.layers.MaxPooling2D(pool_size=(2, 2), padding='same', name='maxpool2D_3')(H)
H = keras.layers.Conv2D(filters=512, kernel_size=3, padding='same', activation='relu', name='conv2d_4_1')(H)
H = keras.layers.Conv2D(filters=512, kernel_size=3, padding='same', activation='relu', name='conv2d_4_2')(H)
H = keras.layers.Conv2D(filters=512, kernel_size=3, padding='same', activation='relu', name='conv2d_4_3')(H)
H = keras.layers.Conv2D(filters=512, kernel_size=3, padding='same', activation='relu', name='conv2d_4_4')(H)

H = keras.layers.BatchNormalization()(H)
H = keras.layers.ReLU()(H)

# 5
H = keras.layers.MaxPooling2D(pool_size=(2, 2), padding='same', name='maxpool2D_4')(H)
H = keras.layers.Conv2D(filters=512, kernel_size=3, padding='same', activation='relu', name='conv2d_5_1')(H)
H = keras.layers.Conv2D(filters=512, kernel_size=3, padding='same', activation='relu', name='conv2d_5_2')(H)
H = keras.layers.Conv2D(filters=512, kernel_size=3, padding='same', activation='relu', name='conv2d_5_3')(H)
H = keras.layers.Conv2D(filters=512, kernel_size=3, padding='same', activation='relu', name='conv2d_5_4')(H)

H = keras.layers.BatchNormalization()(H)
H = keras.layers.ReLU()(H)

# 6
H = keras.layers.MaxPooling2D(pool_size=(2, 2), padding='same', name='maxpool2D_5')(H)
H = keras.layers.Flatten()(H)
H = keras.layers.Dense(units=2048, activation='relu')(H)
H = keras.layers.Dense(units=2048, activation='relu')(H)
Y = keras.layers.Dense(units=num_class, activation='softmax')(H)

model = keras.models.Model(X, Y)
model.compile(
    optimizer=tf.keras.optimizers.Adam(learning_rate=1e-5),
    loss='sparse_categorical_crossentropy',
    metrics=['acc']
)
model.summary()

# val_loss가 제일 적었던 모델을 저장했습니다.

callbacks = tf.keras.callbacks.ModelCheckpoint(filepath='model.h5', monitor='val_loss', save_best_only=True)

start_time = time.time()

history = model.fit(
    train_x, train_y, validation_data=(valid_x, valid_y),
    epochs=50, batch_size=128, callbacks=callbacks
)

print(f'{time.time() - start_time}초 동안 학습함.')

plt.plot(history.history['acc'])
plt.plot(history.history['val_acc'])
plt.title('Model')
plt.xlabel('Epoch')
plt.ylabel('Accuracy')
plt.legend(['Train', 'Test'], loc='upper left')
plt.show()

plt.plot(history.history['loss'])
plt.plot(history.history['val_loss'])
plt.title('Model loss')
plt.xlabel('Epoch')
plt.ylabel('Loss')
plt.legend(['Train', 'Test'], loc='upper left')
plt.show()

model = keras.models.load_model('model.h5')
pred = model.predict(test_x)

plt.figure(figsize=(10,10))
for i in range(25):
    plt.subplot(5, 5, i+1)
    plt.xticks([])
    plt.yticks([])
    plt.grid(False)
    plt.imshow(test_x.reshape(-1, 48, 48)[i], cmap=plt.cm.gray)
    plt.xlabel(f'{idx_to_s[np.argmax(pred[i])]}/{idx_to_s[test_y[i]]}', fontsize=14)
plt.show()
```

