A framework for quickly creating machine learning models using Estimator API of TensorFlow.

<!-- TOC depthFrom:2 depthTo:3 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Installation](#installation)
- [Getting Started](#getting-started)
	- [Example: CNN MNIST Classifier](#example-cnn-mnist-classifier)
	- [Model Function](#model-function)
- [License](#license)

<!-- /TOC -->


## Installation

[Install TensorFlow]:

```sh
pip install tensorflow
```

and run:

```sh
pip install estimator
```

It is recommended to use a [virtual environment].


## Getting Started

```py
from estimator import Model
import tensorflow as tf

# Define the network architecture - layers, number of units, activations etc.
def network(inputs):
    hidden = tf.layers.Dense(units=64, activation=tf.nn.relu)(inputs)
    outputs = tf.layers.Dense(units=10)(hidden)
    return outputs

# Configure the learning process - loss, optimizer, evaluation metrics etc.
model = Model(network,
              loss='sparse_softmax_cross_entropy',
              optimizer=('GradientDescent', 0.001),
              metrics=['accuracy'])

# Train the model using training data
model.train(x_train, y_train, epochs=30, batch_size=128)

# Evaluate the model performance on test or validation data
loss_and_metrics = model.evaluate(x_test, y_test)

# Use the model to make predictions for new data
predictions = model.predict(x)
# or call the model directly
predictions = model(x)
```

More configuration options are available:

```py
model = Model(network,
              loss='sparse_softmax_cross_entropy',
              optimizer=optimizer('GradientDescent', 0.001),
              metrics=['accuracy'],
              model_dir='/tmp/my_model')
```

You can also use custom functions for loss and metrics:

```py
def custom_loss(labels, outputs):
    pass

def custom_metric(labels, outputs):
    pass

model = Model(network,
              loss=custom_loss,
              optimizer=('GradientDescent', 0.001),
              metrics=['accuracy', custom_metric])
```

### Example: CNN MNIST Classifier

This example is based on the [MNIST example] of TensorFlow:

```py
from estimator import Model, GradientDescent, TRAIN
import tensorflow as tf

def network(x, mode):
    x = tf.reshape(x, [-1, 28, 28, 1])
    x = tf.layers.Conv2D(filters=32, kernel_size=[5, 5], padding='same', activation=tf.nn.relu)(x)
    x = tf.layers.MaxPooling2D(pool_size=[2, 2], strides=2)(x)
    x = tf.layers.Conv2D(filters=64, kernel_size=[5, 5], padding='same', activation=tf.nn.relu)(x)
    x = tf.layers.MaxPooling2D(pool_size=[2, 2], strides=2)(x)
    x = tf.layers.Flatten()(x)
    x = tf.layers.Dense(units=1024, activation=tf.nn.relu)(x)
    x = tf.layers.Dropout(rate=0.4)(x, training=mode == TRAIN)
    x = tf.layers.Dense(units=10)(x)
    return x

# Configure the learning process
model = Model(network,
              loss='sparse_softmax_cross_entropy',
              optimizer=('GradientDescent', 0.001))
```

`mode` parameter specifies whether the model is used for training, evaluation or prediction.

### Model Function

To have more control, you may configure the model inside a function using `Estimator` class:

```py
from estimator import Estimator, PREDICT
import tensorflow as tf

def model(features, labels, mode):
    # Define the network architecture
    hidden = tf.layers.Dense(units=64, activation=tf.nn.relu)(features)
    outputs = tf.layers.Dense(units=10)(hidden)
    predictions = tf.argmax(outputs, axis=1)
    # In prediction mode, simply return predictions without configuring learning process
    if mode == PREDICT:
        return predictions

    # Configure the learning process for training and evaluation modes
    loss = tf.losses.sparse_softmax_cross_entropy(labels, outputs)
    optimizer = tf.train.GradientDescentOptimizer(0.001)
    accuracy = tf.metrics.accuracy(labels, predictions)
    return dict(loss=loss,
                optimizer=optimizer,
                metrics={'accuracy': accuracy})

# Create the model using model function
model = Estimator(model)

# Train the model
model.train(x_train, y_train, epochs=30, batch_size=128)
```


## License

[MIT][license]


[license]: /LICENSE
[virtual environment]: https://docs.python.org/3/library/venv.html
[MNIST example]: https://www.tensorflow.org/tutorials/layers#building_the_cnn_mnist_classifier
[Install TensorFlow]: https://www.tensorflow.org/install/
