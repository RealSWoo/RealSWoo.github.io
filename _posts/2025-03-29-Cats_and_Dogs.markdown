---
layout: post
title: Linear Regression
date: 2025-04-05 09:01:00 +0900
category: sample
---

# Cants and Dogs Recogintion
> Code & Graph

This is code
```ruby
# CNN으로 고차원 이미지 분류 : 개, 고양이 이진 분류
import tensorflow as tf
from keras.models import Sequential
from keras.layers import Dense, Conv2D, Flatten, Dropout, MaxPooling2D
from tensorflow.keras.preprocessing.image import ImageDataGenerator
import os
import numpy as np
import matplotlib.pyplot as plt

os.environ["CUDA_VISIBLE_DEVICES"] = ""

# data download
path = '/home/seokwoo/.keras/datasets/cats_and_dogs_filtered'


# 데이터 정리
batch_size = 128
epochs = 15
IMG_HEIGHT = 150
IMG_WIDTH = 150

train_dir = os.path.join(path, 'train')
validation_dir = os.path.join(path, 'validation')
train_cats_dir = os.path.join(train_dir, 'cats')
train_dogs_dir = os.path.join(train_dir, 'dogs')
validation_cats_dir = os.path.join(validation_dir, 'cats')
validation_dogs_dir = os.path.join(validation_dir, 'dogs')

# 이미지 확인
num_cats_tr = len(os.listdir(train_cats_dir))
num_dogs_tr = len(os.listdir(train_dogs_dir))
print(os.listdir(train_cats_dir)[:5])
num_cats_val = len(os.listdir(validation_cats_dir))
num_dogs_val = len(os.listdir(validation_dogs_dir))

total_train = num_cats_tr + num_dogs_tr
total_val = num_cats_val + num_dogs_val

print('total train cat images :', num_cats_tr)
print('total train dog images :', num_dogs_tr)
print('total validation cat images :', num_cats_val)
print('total validation dog images :', num_dogs_val)

print('전체 학습용 데이터 수 :', total_train)
print('전체 검증용 데이터 수 :', total_val)

train_image_gen = ImageDataGenerator(rescale = 1./255) # 하드웨어에 저장
validation_image_gen = ImageDataGenerator(rescale = 1./255)

train_data_gen = train_image_gen.flow_from_directory(batch_size=batch_size, directory=train_dir, shuffle=True,  # 이미지를 128장씩 램에 불러 읽는
                                                     target_size=(IMG_HEIGHT, IMG_WIDTH), class_mode= 'binary') # 크기재조정 라벨링
                                                     #class_mode = 'binary'는 labeling을 해준다.
val_data_gen = validation_image_gen.flow_from_directory(batch_size=batch_size, directory=validation_dir, # validation은 굳이 shuffle 안해도 됌
                                                     target_size=(IMG_HEIGHT, IMG_WIDTH), class_mode= 'binary')

# 데이터 확인 : next() 사용
sample_train_images, _ = next(train_data_gen) #next > 이터러블데이터

def plotImage(img_arr):
  fig, axes = plt.subplots(1, 5, figsize=(10, 10))
  axes = axes.flatten()
  for img, ax in zip(img_arr, axes):
      ax.imshow(img)
      ax.axis('off')
      
  plt.tight_layout()
  plt.show()

plotImage(sample_train_images[:5])


# model
model = Sequential([
    Conv2D(filters=16, kernel_size=3, padding='same', strides=1, activation='relu', input_shape=(IMG_HEIGHT,  IMG_WIDTH, 3)),
    MaxPooling2D(pool_size=2),
    Conv2D(filters=32, kernel_size=3, padding='same', strides=1, activation='relu'),
    MaxPooling2D(pool_size=2),
    Conv2D(filters=64, kernel_size=3, padding='same', strides=1, activation='relu'),
    MaxPooling2D(pool_size=2),
    Flatten(),    
    Dense(units=512, activation='relu'),
    Dense(units=1)
])

model.compile(optimizer = 'adam', loss=tf.keras.losses.BinaryCrossentropy(from_logits=True), metrics=['accuracy'])

print(model.summary())

history = model.fit(
    train_data_gen,
    steps_per_epoch=total_train // batch_size, #에폭당 작업 # 기본값은 len(generator)
    epochs = epochs,
    validation_data = val_data_gen,
    validation_steps=total_train // batch_size
)

#저장
model.save('tf35.hdf5')

# 시각화
acc = history.history['accuracy']
val_acc = history.history['val_accuracy']
loss = history.history['loss']
val_loss = history.history['val_loss']
epochs_range = range(epochs)

plt.figure(figsize=(8, 8))

plt.subplot(1,2,1)
plt.plot(epochs_range, acc, label='Train acc')
# plt.plot(epochs_range, val_acc, label='Train val_acc')
plt.legend(loc='best')
plt.title('Train, Validation acc')

plt.subplot(1,2,2)
plt.plot(epochs_range, loss, label='Train loss')
# plt.plot(epochs_range, val_loss, label='Train val_loss')
plt.legend(loc='best')
plt.title('Train, Validation loss')

plt.show()

# 학습 후 과적합 문제 발생시 : 학습 데이터 수 증가 시키자 - 이미지 보강
image_gen_train = ImageDataGenerator(
    rescale=1./255,
    rotation_range=45,
    width_shift_range=.15,
    height_shift_range=.15,
    horizontal_flip=True,
    zoom_range=0.5
)

train_data_gen = image_gen_train.flow_from_directory(batch_size=batch_size, directory=train_dir, shuffle=True,
                                                     target_size=(IMG_HEIGHT, IMG_WIDTH), class_mode='binary') # (128, 150, 150, 3)
augment_image = [train_data_gen[0][0][1] for i in range(5)]
plotImage(augment_image)

image_gen_val = ImageDataGenerator(rescale=1./255)
val_data_gen = image_gen_val.flow_from_directory(batch_size=batch_size, directory=validation_dir, 
                                                     target_size=(IMG_HEIGHT, IMG_WIDTH), class_mode='binary') # (128, 150, 150, 3)

# model
model_new = Sequential([
    Conv2D(filters=16, kernel_size=3, padding='same', strides=1, activation='relu', input_shape=(IMG_HEIGHT,  IMG_WIDTH, 3)),
    MaxPooling2D(pool_size=2),
    Dropout(rate=0.2),
    Conv2D(filters=32, kernel_size=3, padding='same', strides=1, activation='relu'),
    MaxPooling2D(pool_size=2),
    Conv2D(filters=64, kernel_size=3, padding='same', strides=1, activation='relu'),
    MaxPooling2D(pool_size=2),
    Flatten(),    
    Dense(units=512, activation='relu'),
    Dense(units=1)
])

model_new.compile(optimizer = 'adam', loss=tf.keras.losses.BinaryCrossentropy(from_logits=True), metrics=['accuracy'])

print(model_new.summary())

history = model_new.fit(
    train_data_gen,
    steps_per_epoch=total_train // batch_size, #에폭당 작업 # 기본값은 len(generator)
    epochs = epochs,
    validation_data = val_data_gen,
    validation_steps=total_train // batch_size
)

model_new.save('tf35.hdf5')

# 시각화
acc = history.history['accuracy']
val_acc = history.history['val_accuracy']
loss = history.history['loss']
val_loss = history.history['val_loss']
epochs_range = range(epochs)
plt.figure(figsize=(8, 8))

plt.subplot(1,2,1)
plt.plot(epochs_range, acc, label='Train acc')
# plt.plot(epochs_range, val_acc, label='Train val_acc')
plt.legend(loc='best')
plt.title('Train, Validation acc')

plt.subplot(1,2,2)
plt.plot(epochs_range, loss, label='Train loss')
# plt.plot(epochs_range, val_loss, label='Train val_loss')
plt.legend(loc='best')
plt.title('Train, Validation loss')

plt.show()

import numpy as np
from keras.preprocessing import image

# 파일 경로를 수동으로 입력하거나, 파일 업로드 방식 변경
uploaded_file_path = '/home/seokwoo/testdog.jpg'  # 로컬 파일 경로

# 이미지를 로드하고 크기 조정
img = image.load_img(uploaded_file_path, target_size=(150, 150))

# 이미지를 배열로 변환하고 차원 확장
x = image.img_to_array(img)
x = np.expand_dims(x, axis=0)

# 데이터 정규화: 모델 학습 시 사용한 것과 동일하게 rescale
x = x / 255.0  # 0~1 사이의 값으로 정규화

# 모델 예측
pred = model.predict(x, batch_size=10)
print(pred)
print(pred[0])

# 예측 결과 출력
if pred[0] > 0:
    print(uploaded_file_path + '는 댕댕이')
else:
    print(uploaded_file_path + '는 냥이')
```

These are the results of epochs

['cat.812.jpg', 'cat.278.jpg', 'cat.606.jpg', 'cat.693.jpg', 'cat.14.jpg']   
total train cat images : 1000   
total train dog images : 1000   
total validation cat images : 500   
total validation dog images : 500   
전체 학습용 데이터 수 : 2000   
전체 검증용 데이터 수 : 1000   
Found 2000 images belonging to 2 classes.   
Found 1000 images belonging to 2 classes.   

