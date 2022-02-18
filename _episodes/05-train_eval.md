---
title: "Training and evaluation"
teaching: 20
exercises: 10
questions:
- "What is a convolutional neural network?"
- "What is a saliency map?"
objectives:
- "Train a convolutational neural network for classification."
- "Evalute the network's performance on a test set."
- "Review model performance with saliency maps."
keypoints:
- "Saliency maps can be used to highlight the areas of an image that are discriminative with respect to the given class."
---

## Compile and train your model

Now that the model architecture is complete, it is ready to be compiled and trained! The difference between our predictions and the true values is the eror or "loss". The goal of training is to minimise this loss.

The training process will try to find the optimal model. Using gradient descent, the model's weights are iteratively updated as each batch of data is processed. An epoch means training the neural network with all the training data for one cycle. In an epoch, we use all of the training data once.

```python
# Compile the model defining the 'loss' function type, optimization and the metric.
model.compile(loss='binary_crossentropy', optimizer=custom_adam, metrics=['acc'])

# Save the best model found during training
checkpointer = ModelCheckpoint(filepath='best_model.hdf5', monitor='val_loss',
                               verbose=1, save_best_only=True)

# Finally let's train our network
steps_per_epoch = len(dataset_train)//batch_size
hist = model.fit_generator(datagen.flow(dataset_train, labels_train, batch_size=16), 
                                     steps_per_epoch=steps_per_epoch, 
                                     epochs=10, 
                                     validation_data= (dataset_val, labels_val), 
                                     callbacks=[checkpointer])
```
{: .language-python}

We can now plot the results of the training. Our hope is that "loss" drops over successive epochs, while accuracy increases.

```python
plt.plot(hist.history['loss'], 'b-', label='train loss')
plt.plot(hist.history['val_loss'], 'r-', label='val loss')
plt.ylabel('Loss')
plt.xlabel('Epoch')
plt.legend(loc='lower right')
plt.show()

plt.plot(hist.history['acc'], 'b-', label='train accuracy')
plt.plot(hist.history['val_acc'], 'r-', label='val accuracy')
plt.ylabel('Accuracy')
plt.xlabel('Epoch')
plt.legend(loc='lower right')
plt.show()
```
{: .language-python}

![Training curves](../fig/placeholder.png){: width="600px"}

## Evaluating your model on the held-out test set

In this step, we will present the entire test dataset for the model we created, in order to calculate the accuracy of our neural network in a group of images that the model has never seen before.

```python
#Let's import the keras function that opens previously saved models
from keras.models import load_model 

# We open the best model that we saved in training
best_model = load_model('best_model.hdf5') 

print ('\nNeural network weights updated to the best epoch.')
```
{: .language-python}

Now that we've loaded the best model, we can evaluate the accuracy on our test data.

```python
# We use the evaluate function to evaluate the accuracy of our model in the test group
print(f"Accuracy in test group: {best_model.evaluate(dataset_test, labels_test, verbose=0)[1]}")
```
{: .language-python}

## Saliency maps

Saliency maps are used to rank the pixels in an image based on their contribution to the networks prediction. Here we compute saliency maps that highlight the areas of an image that are discriminative with respect to the given class.

```python
# !pip install tf_keras_vis
from matplotlib import cm
from tf_keras_vis.gradcam import Gradcam

import numpy as np
from matplotlib import pyplot as plt
from tf_keras_vis.gradcam_plus_plus import GradcamPlusPlus
from tf_keras_vis.utils.scores import CategoricalScore

gradcam = GradcamPlusPlus(best_model,
                          clone=True)

def plot_map(cam, classe, prediction, img):
    fig, axes = plt.subplots(1,2,figsize=(14,5))
    axes[0].imshow(np.squeeze(img), cmap='gray')
    axes[1].imshow(np.squeeze(img), cmap='gray')
    heatmap = np.uint8(cm.jet(cam[0])[..., :3] * 255)
    i = axes[1].imshow(heatmap,cmap="jet",alpha=0.5)
    fig.colorbar(i)
    plt.suptitle("Class: {}. Pred = {}".format(classe, prediction))
    
for image_id in range(10):
    
    SEED_INPUT = dataset_test[image_id]
    CATEGORICAL_INDEX = [0]

    layer_idx = 18
    penultimate_layer_idx = 13
    class_idx  = 0

    cat_score = labels_test[image_id]
    
    cat_score = CategoricalScore(CATEGORICAL_INDEX)
    cam = gradcam(cat_score, SEED_INPUT, 
                  penultimate_layer = penultimate_layer_idx,
                  normalize_cam=True)
    
    #Let's show which class it belongs to
    _classe = 'normal' if labels_test[image_id]==0 else 'effusion'

    _prediction = best_model.predict(dataset_test[image_id][np.newaxis,:,...], verbose=0)

    plot_map(cam, _classe, _prediction[0][0], SEED_INPUT)
```
{: .language-python}

![Saliency maps](../fig/placeholder.png){: width="600px"}

{% include links.md %}
 


