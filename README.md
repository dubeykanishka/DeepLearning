# DeepLearning

This is the project details, my code is in the other folder

{
  "nbformat": 4,
  "nbformat_minor": 0,
  "metadata": {
    "accelerator": "GPU",
    "colab": {
      "name": "Copy of Copy of autoencoder.ipynb",
      "provenance": [],
      "private_outputs": true,
      "collapsed_sections": [],
      "toc_visible": true,
      "include_colab_link": true
    },
    "kernelspec": {
      "display_name": "Python 3",
      "language": "python",
      "name": "python3"
    }
  },
  "cells": [
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "view-in-github",
        "colab_type": "text"
      },
      "source": [
        "<a href=\"https://colab.research.google.com/github/gmprovan/CS6421-Assignment1/blob/master/Copy_of_Copy_of_autoencoder.ipynb\" target=\"_parent\"><img src=\"https://colab.research.google.com/assets/colab-badge.svg\" alt=\"Open In Colab\"/></a>"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "7H3yTncQfoym"
      },
      "source": [
        "##### Copyright 2019 The TensorFlow Authors."
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "cellView": "form",
        "colab_type": "code",
        "id": "h1CiDh7CfqON",
        "colab": {}
      },
      "source": [
        "#@title Licensed under the Apache License, Version 2.0 (the \"License\");\n",
        "# you may not use this file except in compliance with the License.\n",
        "# You may obtain a copy of the License at\n",
        "#\n",
        "# https://www.apache.org/licenses/LICENSE-2.0\n",
        "#\n",
        "# Unless required by applicable law or agreed to in writing, software\n",
        "# distributed under the License is distributed on an \"AS IS\" BASIS,\n",
        "# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.\n",
        "# See the License for the specific language governing permissions and\n",
        "# limitations under the License."
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "pp-UaomMQJNo"
      },
      "source": [
        "# CS6421 Final Project: Deep Learning\n",
        "\n",
        "\n",
        "**Implementing Real-World examples**\n",
        "\n",
        "This assignment is the core programming assignment for Deep Learning (CS6421). You will be given a tutorial introduction to the deep autoencoder, and will then need to use this model to solve two real-world problems:\n",
        "- text noise removal\n",
        "- pedestrian safety analysis on footpaths\n",
        "\n",
        "If you follow the code provided you should be able to score well. To obtain top marks you will need to extend the provided code fragments with high-performing models that show what you have learned in the course.\n",
        "\n",
        "The marks for the project are as follows:\n",
        "- 1: extensions to basic autoencoder [10 marks]\n",
        "- 2: extensions to de-noising autoencoder [10 marks]\n",
        "- 3: text reconstruction model and results [40 marks]\n",
        " \n",
        "Please submit your work ideally in a clear Jupyter notebook, highlighting the code that you have written. Present the comparisons of different model performance results in clear tables. Alternatively, you can creat a pdf document that summarises all of this. In any event, I will need a Jupyter notebook that I can run if I have any queries about your work. If I cannot compile any code submitted then you will get 0 for the results obtained for that code."
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "wLASBMNLsUe6",
        "colab_type": "text"
      },
      "source": [
        "For each assignment, the following text needs to be attached and agreed to:\n",
        "\n",
        " By submitting this exam, I declare\n",
        "\n",
        "(1) that all work of it is my own;\n",
        "\n",
        "(2) that I did not seek whole or partial solutions for any part of my submission from others; and\n",
        "\n",
        "(3) that I did not and will not discuss, exchange, share, or publish complete or partial solutions for this exam or any part of it."
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "NBhbug0Qgmuu"
      },
      "source": [
        "## Introduction: Basic Autoencoder\n",
        "In this assignment, we will create a **simple autoencoder** model using the [TensorFlow subclassing API](https://www.tensorflow.org/guide/keras#model_subclassing). We start with the popular [MNIST dataset](http://yann.lecun.com/exdb/mnist/) (Grayscale images of hand-written digits from 0 to 9).\n",
        "_[This first section is based on a notebook orignially contributed by: [afagarap](https://github.com/afagarap)]_\n",
        "\n",
        "\"Autoencoding\" is a data compression algorithm where the compression and decompression functions are 1) data-specific, 2) lossy, and 3) learned automatically from examples rather than engineered by a human. Additionally, in almost all contexts where the term \"autoencoder\" is used, the compression and decompression functions are implemented with neural networks.\n",
        " \n",
        " 1) Autoencoders are _data-specific_, which means that they will only be able to compress data similar to what they have been trained on. This is different from, say, the MPEG-2 Audio Layer III (MP3) compression algorithm, which only holds assumptions about \"sound\" in general, but not about specific types of sounds. An autoencoder trained on pictures of faces would do a rather poor job of compressing pictures of trees, because the features it would learn would be face-specific.\n",
        "\n",
        "2) Autoencoders are _lossy_, which means that the decompressed outputs will be degraded compared to the original inputs (similar to MP3 or JPEG compression). This differs from lossless arithmetic compression.\n",
        "\n",
        "3) Autoencoders are _learned automatically from data examples_, which is a useful property: it means that it is easy to train specialized instances of the algorithm that will perform well on a specific type of input. It doesn't require any new engineering, just appropriate training data.\n",
        "\n",
        "To build an autoencoder, you need three things: an encoding function, a decoding function, and a distance function between the amount of information loss between the compressed representation of your data and the decompressed representation (i.e. a \"loss\" function). The encoder and decoder will be chosen to be parametric functions (typically neural networks), and to be differentiable with respect to the distance function, so the parameters of the encoding/decoding functions can be optimize to minimize the reconstruction loss, using Stochastic Gradient Descent. \n",
        "\n",
        "In general, a neural network is a computational model that is used for finding a function describing the relationship between data features $x$ and its values or labels $y$, i.e. $y = f(x)$. \n",
        "An autoencoder is specific type of neural network, which consists of encoder and decoder components: (1) the **encoder**, which learns a compressed data representation $z$, and (2) the **decoder**, which reconstructs the data $\\hat{x}$ based on its idea $z$ of how it is structured:\n",
        "$$ z = f\\big(h_{e}(x)\\big)$$\n",
        "$$ \\hat{x} = f\\big(h_{d}(z)\\big),$$\n",
        "where $z$ is the learned data representation by encoder $h_{e}$, and $\\hat{x}$ is the reconstructed data by decoder $h_{d}$ based on $z$."
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "zK2SZnszhSYk"
      },
      "source": [
        "## Setup\n",
        "We start by importing the libraries and functions that we will need."
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "S1sepk9uMddm",
        "colab": {}
      },
      "source": [
        "from __future__ import absolute_import, division, print_function, unicode_literals\n",
        "\n",
        "#try:\n",
        "  # The %tensorflow_version magic only works in colab.\n",
        "  # tensorflow_version 2.x\n",
        "#except Exception:\n",
        "#  pass\n",
        "import numpy as np\n",
        "import matplotlib.pyplot as plt\n",
        "import tensorflow as tf\n",
        "\n",
        "from tensorflow.keras.datasets import mnist\n",
        "\n",
        "print('TensorFlow version:', tf.__version__)\n",
        "print('Is Executing Eagerly?', tf.executing_eagerly())"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "CH3QqWQ1RiP2"
      },
      "source": [
        "## Autoencoder model\n",
        "\n",
        "The encoder and decoder are defined as:\n",
        "$$ z = f\\big(h_{e}(x)\\big)$$\n",
        "$$ \\hat{x} = f\\big(h_{d}(z)\\big),$$\n",
        "where $z$ is the compressed data representation generated by encoder $h_{e}$, and $\\hat{x}$ is the reconstructed data generated by decoder $h_{d}$ based on $z$.\n",
        "\n",
        "<div align=\"center\"><img src=\"https://github.com/benjaminirving/mlseminars-autoencoders/blob/master/imgs/d1.png?raw=1\" width=\"80%\"></div>\n",
        "\n",
        "In this figure, we take as input an image, and compress that image before decompressing it using a Dense network. We further define a simple model for this below."
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "fiYJ9BLW-w40"
      },
      "source": [
        "### Define an encoder layer\n",
        "\n",
        "The first component, the **encoder**, is similar to a conventional feed-forward network. However, it's function is not  predicting values (a _regression_ task) or categories (a _classification_ task). Instead, it's function is to learn a compressed data structure  $z$. We can implement the encoder layer as dense layers, as follows:"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "fdl5_0Z4-w41",
        "colab": {}
      },
      "source": [
        "class Encoder(tf.keras.layers.Layer):\n",
        "    def __init__(self, intermediate_dim):\n",
        "        super(Encoder, self).__init__()\n",
        "        self.hidden_layer = tf.keras.layers.Dense(units=intermediate_dim, activation=tf.nn.relu)\n",
        "        self.output_layer = tf.keras.layers.Dense(units=intermediate_dim, activation=tf.nn.relu)\n",
        "    \n",
        "    def call(self, input_features):\n",
        "        activation = self.hidden_layer(input_features)\n",
        "        return self.output_layer(activation)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "7WgR-V0--w44"
      },
      "source": [
        "The _encoding_ is done by passing data input $x$ to the encoder's hidden layer $h$ in order to learn the data representation $z = f(h(x))$.\n",
        "\n",
        "We first create an `Encoder` class that inherits the [`tf.keras.layers.Layer`](https://www.tensorflow.org/api_docs/python/tf/keras/layers/Layer) class to define it as a layer. The compressed layer $z$ is a _component_ of the autoencoder model.\n",
        "\n",
        "Analyzing the code, the `Encoder` layer is defined to have a single hidden layer of neurons (`self.hidden_layer`) to learn the input features. Then, we connect the hidden layer to a layer (`self.output_layer`) that encodes the learned activations to the lower dimensional layer for $z$. "
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "W9me1ZLq-w45"
      },
      "source": [
        "### Define a decoder layer\n",
        "\n",
        "The second component, the **decoder**, is also similar to a feed-forward network. However, instead of reducing data to lower dimension, it attempts to reverse the process, i.e. reconstruct the data $\\hat{x}$ from its lower dimension representation $z$ to its original dimension.\n",
        "\n",
        "The _decoding_ is done by passing the lower dimension representation $z$ to the decoder's hidden layer $h$ in order to reconstruct the data to its original dimension $\\hat{x} = f(h(z))$. We can implement the decoder layer as follows,"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "m3za66lwMjWX",
        "colab": {}
      },
      "source": [
        "class Decoder(tf.keras.layers.Layer):\n",
        "    def __init__(self, intermediate_dim, original_dim):\n",
        "        super(Decoder, self).__init__()\n",
        "        self.hidden_layer = tf.keras.layers.Dense(units=intermediate_dim, activation=tf.nn.relu)\n",
        "        self.output_layer = tf.keras.layers.Dense(units=original_dim, activation=tf.nn.relu)\n",
        "  \n",
        "    def call(self, code):\n",
        "        activation = self.hidden_layer(code)\n",
        "        return self.output_layer(activation)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "SU-QtEUo-w4-"
      },
      "source": [
        "We now create a `Decoder` class that also inherits the `tf.keras.layers.Layer`.\n",
        "\n",
        "The `Decoder` layer is also defined to have a single hidden layer of neurons to reconstruct the input features $\\hat{x}$ from the learned representation $z$ by the encoder $f\\big(h_{e}(x)\\big)$. Then, we connect its hidden layer to a layer that decodes the data representation from lower dimension $z$ to its original dimension $\\hat{x}$. Hence, the \"output\" of the `Decoder` layer is the reconstructed data $\\hat{x}$ from the data representation $z$.\n",
        "\n",
        "Ultimately, the output of the decoder is the autoencoder's output.\n",
        "\n",
        "Now that we have defined the components of our autoencoder, we can finally build our model."
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "uZRIHpkl-w5A"
      },
      "source": [
        "### Build the autoencoder model\n",
        "\n",
        "We can now build the autoencoder model by instantiating `Encoder` and `Decoder` layers."
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "Rl5HUez7-w5C",
        "colab": {}
      },
      "source": [
        "class Autoencoder(tf.keras.Model):\n",
        "  def __init__(self, intermediate_dim, original_dim):\n",
        "    super(Autoencoder, self).__init__()\n",
        "    self.loss = []\n",
        "    self.encoder = Encoder(intermediate_dim=intermediate_dim)\n",
        "    self.decoder = Decoder(intermediate_dim=intermediate_dim, original_dim=original_dim)\n",
        "\n",
        "  def call(self, input_features):\n",
        "    code = self.encoder(input_features)\n",
        "    reconstructed = self.decoder(code)\n",
        "    return reconstructed"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "25zYoLju-w5G"
      },
      "source": [
        "As discussed above, the encoder's output is the input to the decoder, as it is written above (`reconstructed = self.decoder(code)`)."
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "X9NBnWhSnvn4"
      },
      "source": [
        "## Reconstruction error\n",
        "\n",
        "To learn the compressed layer $z$, we define a loss function over the difference between the input data $x$ and the reconstruction of $x$, which is $\\hat{x}$.\n",
        "We call this comparison the reconstruction error function, a given by the following equation:\n",
        "$$ L = \\dfrac{1}{n} \\sum_{i=0}^{n-1} \\big(\\hat{x}_{i} - x_{i}\\big)^{2}$$\n",
        "where $\\hat{x}$ is the reconstructed data while $x$ is the original data."
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "wmWZG-1qLP8p",
        "colab": {}
      },
      "source": [
        "def loss(preds, real):\n",
        "  return tf.reduce_mean(tf.square(tf.subtract(preds, real)))"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "KVjT6C2SqmH0"
      },
      "source": [
        "## Forward pass and optimization\n",
        "\n",
        "We will write a function for computing the forward pass, and applying a chosen optimization function."
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "KNSJqjY7qnCe",
        "colab": {}
      },
      "source": [
        "def train(loss, model, opt, original):\n",
        "  with tf.GradientTape() as tape:\n",
        "    preds = model(original)\n",
        "    reconstruction_error = loss(preds, original)\n",
        "  gradients = tape.gradient(reconstruction_error, model.trainable_variables)\n",
        "  gradient_variables = zip(gradients, model.trainable_variables)\n",
        "  opt.apply_gradients(gradient_variables)\n",
        "  \n",
        "  return reconstruction_error"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "8c4bNlGLrd5T"
      },
      "source": [
        "## The training loop\n",
        "\n",
        "Finally, we will write a function to run the training loop. This function will take arguments for the model, the optimization function, the loss, the dataset, and the training epochs.\n",
        "\n",
        "The training loop itself uses a `GradientTape` context defined in `train` for each batch."
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "K8Sh1UaQrc5D",
        "colab": {}
      },
      "source": [
        "def train_loop(model, opt, loss, dataset, epochs):\n",
        "  for epoch in range(epochs):\n",
        "    epoch_loss = 0\n",
        "    for step, batch_features in enumerate(dataset):\n",
        "      loss_values = train(loss, model, opt, batch_features)\n",
        "      epoch_loss += loss_values\n",
        "    model.loss.append(epoch_loss)\n",
        "    print('Epoch {}/{}. Loss: {}'.format(epoch + 1, epochs, epoch_loss.numpy()))"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "8eCxbz9ZSwjr"
      },
      "source": [
        "## Process the dataset\n",
        "\n",
        "Now that we have defined our `Autoencoder` class, the loss function, and the training loop, let's import the dataset. We will normalize the pixel values for each example through dividing by maximum pixel value. We shall flatten the examples from 28 by 28 arrays to 784-dimensional vectors."
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "eAN1ONp6MvI7",
        "colab": {}
      },
      "source": [
        "from tensorflow.keras.datasets import mnist\n",
        "(x_train, _), (x_test, _) = mnist.load_data()\n",
        "\n",
        "x_train = x_train / 255.\n",
        "\n",
        "x_train = x_train.astype(np.float32)\n",
        "x_train = np.reshape(x_train, (x_train.shape[0], 784))\n",
        "x_test = np.reshape(x_test, (x_test.shape[0], 784))\n",
        "\n",
        "training_dataset = tf.data.Dataset.from_tensor_slices(x_train).batch(256)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "268qdJGGTULP"
      },
      "source": [
        "## Train the model\n",
        "\n",
        "Now all we have to do is instantiate the autoencoder model and choose an optimization function, then pass the intermediate dimension and the original dimension of the images."
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "t8nw7mdKMxvb",
        "colab": {}
      },
      "source": [
        "model = Autoencoder(intermediate_dim=128, original_dim=784)\n",
        "opt = tf.keras.optimizers.Adam(learning_rate=1e-2)\n",
        "\n",
        "train_loop(model, opt, loss, training_dataset, 20)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "VioflTOhTnwl"
      },
      "source": [
        "## Plot the in-training performance\n",
        "\n",
        "Let's take a look at how the model performed during training in a couple of plots."
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "azgmhikhM0EE",
        "colab": {}
      },
      "source": [
        "plt.plot(range(20), model.loss)\n",
        "plt.xlabel('Epochs')\n",
        "plt.ylabel('Loss')\n",
        "plt.show()"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "JccKWvNYTtUW"
      },
      "source": [
        "## Predictions\n",
        "\n",
        "Finally, we will look at some of the predictions. The wrong predictions are labeled in red."
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "colab_type": "code",
        "id": "-JwpigAlM2dF",
        "colab": {}
      },
      "source": [
        "number = 10  # how many digits we will display\n",
        "plt.figure(figsize=(20, 4))\n",
        "for index in range(number):\n",
        "    # display original\n",
        "    ax = plt.subplot(2, number, index + 1)\n",
        "    plt.imshow(x_test[index].reshape(28, 28))\n",
        "    plt.gray()\n",
        "    ax.get_xaxis().set_visible(False)\n",
        "    ax.get_yaxis().set_visible(False)\n",
        "\n",
        "    # display reconstruction\n",
        "    ax = plt.subplot(2, number, index + 1 + number)\n",
        "    plt.imshow(model(x_test)[index].numpy().reshape(28, 28))\n",
        "    plt.gray()\n",
        "    ax.get_xaxis().set_visible(False)\n",
        "    ax.get_yaxis().set_visible(False)\n",
        "plt.show()"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "colab_type": "text",
        "id": "o4rkBaWN-w5r"
      },
      "source": [
        "## 1. Tasks for Basic Autoencoder Assignment\n",
        "\n",
        "As you may see after training this model, the reconstructed images are quite blurry. A number of things could be done to move forward from this point, e.g. adding more layers, or using a convolutional neural network architecture as the basis of the autoencoder, or use a different kind of autoencoder.\n",
        "\n",
        "- generate results for 3 different Dense architectures and summarise the impact of architecture on performance\n",
        "- define 2 CNN  architectures (similar to as described) and compare their performance to that of the Dense models\n",
        "\n",
        "\n",
        "\n",
        "\n",
        "\n",
        "\n",
        "\n",
        "\n"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "28bbYL20Lrss",
        "colab_type": "text"
      },
      "source": [
        "Since our inputs are images, it makes sense to use convolutional neural networks (CNNs) as encoders and decoders. In practical settings, autoencoders applied to images are always convolutional autoencoders -- they simply perform much better.\n",
        "\n",
        "- implement a CNN model, where the encoder will consist of a stack of Conv2D and MaxPooling2D layers (max pooling being used for spatial down-sampling), while the decoder will consist in a stack of Conv2D and UpSampling2D layers. To improve the quality of the reconstructed image, we use more filters per layer. The model details are:"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "21ArKpJSoa16",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "input_img = tf.keras.layers.Input(shape=(28, 28, 1)) # adapt this if using `channels_first` image data format\n",
        "\n",
        "x = tf.keras.layers.Conv2D(16, (3, 3), activation='relu', padding='same')(input_img)\n",
        "x = tf.keras.layers.MaxPooling2D((2, 2), padding='same')(x)\n",
        "x = tf.keras.layers.Conv2D(8, (3, 3), activation='relu', padding='same')(x)\n",
        "x = tf.keras.layers.MaxPooling2D((2, 2), padding='same')(x)\n",
        "x = tf.keras.layers.Conv2D(8, (3, 3), activation='relu', padding='same')(x)\n",
        "encoded = tf.keras.layers.MaxPooling2D((2, 2), padding='same')(x)\n",
        "\n",
        "# at this point the representation is (4, 4, 8) i.e. 128-dimensional\n",
        "\n",
        "x = tf.keras.layers.Conv2D(8, (3, 3), activation='relu', padding='same')(encoded)\n",
        "x = tf.keras.layers.UpSampling2D((2, 2))(x)\n",
        "x = tf.keras.layers.Conv2D(8, (3, 3), activation='relu', padding='same')(x)\n",
        "x = tf.keras.layers.UpSampling2D((2, 2))(x)\n",
        "x = tf.keras.layers.Conv2D(16, (3, 3), activation='relu')(x)\n",
        "x = tf.keras.layers.UpSampling2D((2, 2))(x)\n",
        "decoded = tf.keras.layers.Conv2D(1, (3, 3), activation='sigmoid', padding='same')(x)\n",
        "\n",
        "autoencoder = tf.keras.models.Model(input_img, decoded)\n",
        "autoencoder.compile(optimizer='adadelta', loss='binary_crossentropy')"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "ysxpFS5Rl-Jj",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# to train this model we will with original MNIST digits with shape (samples, 3, 28, 28) and we will just normalize pixel values between 0 and 1\n",
        "# (x_train, _), (x_test, _) = load_data('../input/mnist.npz')\n",
        "from tensorflow.keras.datasets import mnist\n",
        "(x_train, _), (x_test, _) = mnist.load_data()\n",
        "\n",
        "x_train = x_train.astype('float32') / 255.\n",
        "x_test = x_test.astype('float32') / 255.\n",
        "x_train = np.reshape(x_train, (len(x_train), 28, 28, 1))\n",
        "x_test = np.reshape(x_test, (len(x_test), 28, 28, 1))"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "rBiU7jW9mAdG",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "autoencoder.fit(x_train, x_train, epochs=50, batch_size=128, \n",
        "                shuffle=True, validation_data=(x_test, x_test), \n",
        "                callbacks=[tf.keras.callbacks.TensorBoard(log_dir='./tmp/autoencoder')])"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "qsRzfT-2LjxN",
        "colab_type": "text"
      },
      "source": [
        "\n",
        "- train the model for 100 epochs and compare the results to the model where you use a dense encoding rather than convolutions.\n",
        "\n",
        "**Scoring**: 10 marks total\n",
        "- models [5 marks]: \n",
        "  - Dense, multi-layer model [given];\n",
        "  - CNN basic model [given];\n",
        "  - CNN complex model [5 marks].\n",
        "- results and discussion [5 marks]: present the results in a clear fashion and explain why the results were as obtained. Good experimental design and statistical significance testing will be rewarded."
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "4rmry8GsoFfi",
        "colab_type": "text"
      },
      "source": [
        "## 2. Denoising autoencoder\n",
        "\n",
        "For this real-world application, we will use an autoencoder to remove noise from an image. To do this, we\n",
        "\n",
        "- learn a more robust representation by forcing the autoencoder to learn an input from a corrupted version of itself\n",
        "\n",
        "<div align=\"center\"><img src=\"https://github.com/benjaminirving/mlseminars-autoencoders/blob/master/imgs/d3.png?raw=1\" width=\"60%\"></div>\n",
        "\n",
        "The first step: generate synthetic noisy digits as follows: apply a gaussian noise matrix and clip the images between 0 and 1.\n",
        "\n",
        "\n"
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "WbtNDoNhot-J",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "from keras.datasets import mnist\n",
        "import numpy as np\n",
        "\n",
        "(x_train, _), (x_test, _) = mnist.load_data()\n",
        "\n",
        "x_train = x_train.astype('float32') / 255.\n",
        "x_test = x_test.astype('float32') / 255.\n",
        "x_train = np.reshape(x_train, (len(x_train), 28, 28, 1))\n",
        "x_test = np.reshape(x_test, (len(x_test), 28, 28, 1))\n",
        "\n",
        "# Introduce noise with a probability factor of 0.5\n",
        "noise_factor = 0.5\n",
        "x_train_noisy = x_train + noise_factor + np.random.normal(loc=0.0, scale=1.0, size=x_train.shape)\n",
        "x_test_noisy = x_test + noise_factor + np.random.normal(loc=0.0, scale=1.0, size=x_test.shape)\n",
        "\n",
        "x_train_noisy = np.clip(x_train_noisy, 0., 1.)\n",
        "x_test_noisy = np.clip(x_test_noisy, 0., 1.)"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "Qm59eENMpCeC",
        "colab_type": "text"
      },
      "source": [
        "Next, plot some figures to see what the digits look like with noise added."
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "3tnUZbmOpAiz",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Plot figures to show what the noisy digits look like\n",
        "n = 10\n",
        "plt.figure(figsize=(20, 2))\n",
        "for i in range(n):\n",
        "    ax = plt.subplot(1, n, i + 1)\n",
        "    plt.imshow(x_test_noisy[i].reshape(28, 28))\n",
        "    plt.gray()\n",
        "    ax.get_xaxis().set_visible(False)\n",
        "    ax.get_yaxis().set_visible(False)\n",
        "plt.show()"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "luZuk1rVpakh",
        "colab_type": "text"
      },
      "source": [
        "- Train the model for 100 epochs and compare the results to the model where you use a dense encoding rather than convolutions."
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "7DvL3Q_gpfzm",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# This will train for 100 epochs\n",
        "autoencoder.fit(x_train_noisy, x_train, epochs=100, batch_size=128, \n",
        "                shuffle=True, validation_data=(x_test_noisy, x_test), \n",
        "                callbacks=[tf.keras.callbacks.TensorBoard(log_dir='./tmp/tb', histogram_freq=0, write_graph=False)])\n"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "RySbrS-zssGb",
        "colab_type": "text"
      },
      "source": [
        "**Scoring**: 10 marks total\n",
        "- models [5 marks]: \n",
        "  - Dense, multi-layer model [given];\n",
        "  - CNN basic model [given];\n",
        "  - CNN complex model [5 marks].\n",
        "- results and discussion [5 marks]: present the results in a clear fashion and explain why the results were as obtained. Good experimental design and statistical significance testing will be rewarded."
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "fU7s_1p-RGWR",
        "colab_type": "text"
      },
      "source": [
        "**3. Text Reconstruction Application**\n",
        "\n",
        "You will now use the approach just described to reconstruct corrupted text. We will use a small dataset with grey-scale images of size $420 \\times 540$.  The steps required are as follows:\n",
        "\n",
        "- Apply this autoencoder approach (as just described) to the text data provided as noted below.\n",
        "- The data has two sets of images, train (https://github.com/gmprovan/CS6421-Assignment1/blob/master/train.zip) and test (https://github.com/gmprovan/CS6421-Assignment1/blob/master/test.zip). These images contain various styles of text, to which synthetic noise has been added to simulate real-world, messy artifacts. The training set includes the test without the noise (train_cleaned: https://github.com/gmprovan/CS6421-Assignment1/blob/master/train_cleaned.zip). \n",
        "- You must create an algorithm to clean the images in the test set, and report the error as RMSE (root-mean-square error).\n",
        "\n",
        "**Scoring**: 40 marks total\n",
        "- models [25 marks]: \n",
        "  - Dense, multi-layer model [5 marks];\n",
        "  - CNN basic model [5 marks];\n",
        "  - CNN complex models (at least 2) [15 marks].\n",
        "- results and discussion [15 marks]: present the results in a clear fashion and explain why the results were as obtained. Good experimental design and statistical significance testing will be rewarded.\n",
        "\n",
        "\n",
        "you will get full marks if you achieve RMSE < 0.005. Deductions are as follows:\n",
        "- -1: 0.01 $\\leq$ RMSE $\\leq$ 0.005\n",
        "- -5: 0.05 $\\leq$ RMSE $\\leq$ 0.01\n",
        "- -10:  RMSE $>$ 0.05\n"
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "N1Q4N4WFcxp_",
        "colab_type": "text"
      },
      "source": [
        "The data should have been properly pre-processed already. If you want to pre-process the image data more, use the code environments below (e.g., skimage, keras.preprocessing.image), and then plot some samples of data."
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "9OcCACBMcut4",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# This Python 3 environment comes with many helpful analytics libraries installed\n",
        "# It is defined by the kaggle/python docker image: https://github.com/kaggle/docker-python\n",
        "# For example, here's several helpful packages to load in \n",
        "import os\n",
        "from pathlib import Path\n",
        "import glob\n",
        "import numpy as np # linear algebra\n",
        "import pandas as pd # data processing, CSV file I/O (e.g. pd.read_csv)\n",
        "import matplotlib.pyplot as plt\n",
        "from skimage.io import imread, imshow, imsave\n",
        "from keras.preprocessing.image import load_img, array_to_img, img_to_array\n",
        "from keras.models import Sequential, Model\n",
        "from keras.layers import Dense, Conv2D, MaxPooling2D, UpSampling2D, Flatten, Input\n",
        "from keras.optimizers import SGD, Adam, Adadelta, Adagrad\n",
        "from keras import backend as K\n",
        "from sklearn.model_selection import train_test_split\n",
        "np.random.seed(111)\n",
        "# Input data files are available in the \"../input/\" directory.\n",
        "# For example, running this (by clicking run or pressing Shift+Enter) will list the files in the input directory\n",
        "   \n"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "KIvKDS0xd1aL",
        "colab_type": "text"
      },
      "source": [
        "Next you must build the model. I provide the code framework, with the model details left up to you."
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "rjx_N3Rnd8fF",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Lets' define our autoencoder now\n",
        "def build_autoenocder():\n",
        "    input_img = Input(shape=(420,540,1), name='image_input')\n",
        "    \n",
        "    #enoder \n",
        "    # enter encoder model here\n",
        "    \n",
        "    #decoder\n",
        "    # enter decoder model model\n",
        " \n",
        "    \n",
        "    #model\n",
        "    autoencoder = Model(inputs=input_img, outputs=x)\n",
        "    autoencoder.compile(optimizer='adam', loss='binary_crossentropy')\n",
        "    return autoencoder\n",
        "autoencoder = build_autoenocder()\n",
        "autoencoder.summary()\n"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "aVXVNyo5fBMu",
        "colab_type": "text"
      },
      "source": [
        "Next we have code to compile and run the model. Please modify the code below to fit your purposes."
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "LR60krHYfEgj",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "X = []\n",
        "Y = []\n",
        "\n",
        "for img in train_images:\n",
        "    img = load_img(train / img, grayscale=True,target_size=(420,540))\n",
        "    img = img_to_array(img).astype('float32')/255.\n",
        "    X.append(img)\n",
        "\n",
        "for img in train_labels:\n",
        "    img = load_img(train_cleaned / img, grayscale=True,target_size=(420,540))\n",
        "    img = img_to_array(img).astype('float32')/255.\n",
        "    Y.append(img)\n",
        "\n",
        "\n",
        "X = np.array(X)\n",
        "Y = np.array(Y)\n",
        "\n",
        "print(\"Size of X : \", X.shape)\n",
        "print(\"Size of Y : \", Y.shape)\n",
        "\n",
        "# Split the dataset into training and validation. Always set the random state!!\n",
        "X_train, X_valid, y_train, y_valid = train_test_split(X, Y, test_size=0.1, random_state=111)\n",
        "print(\"Total number of training samples: \", X_train.shape)\n",
        "print(\"Total number of validation samples: \", X_valid.shape)\n",
        "\n",
        "# Train your model\n",
        "autoencoder.fit(X_train, y_train, epochs=10, batch_size=8, validation_data=(X_valid, y_valid))\n"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "2LR7l0fjfVnv",
        "colab_type": "text"
      },
      "source": [
        "Next we compute the predictions from the trained model. Again, modify the code structure below as necessary."
      ]
    },
    {
      "cell_type": "code",
      "metadata": {
        "id": "3-IkMCkPfZJm",
        "colab_type": "code",
        "colab": {}
      },
      "source": [
        "# Compute the prediction\n",
        "predicted_label = np.squeeze(autoencoder.predict(sample_test_img))\n",
        "\n",
        "f, ax = plt.subplots(1,2, figsize=(10,8))\n",
        "ax[0].imshow(np.squeeze(sample_test), cmap='gray')\n",
        "ax[1].imshow(np.squeeze(predicted_label.astype('int8')), cmap='gray')\n",
        "plt.show()\n"
      ],
      "execution_count": 0,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "UZNhwBltfjXz",
        "colab_type": "text"
      },
      "source": [
        "To complete this part of the project, you need to examine methods to improve the quality of the predictions. Report on methods to get better performance on this task."
      ]
    },
    {
      "cell_type": "markdown",
      "metadata": {
        "id": "lXI9G4whP-sV",
        "colab_type": "text"
      },
      "source": [
        "## References\n",
        "* Martín Abadi, Ashish Agarwal, Paul Barham, Eugene Brevdo, Zhifeng Chen, Craig Citro, Greg S. Corrado, Andy Davis, Jeffrey Dean, Matthieu Devin, Sanjay Ghemawat, Ian Goodfellow, Andrew Harp, Geoffrey Irving, Michael Isard, Rafal Jozefowicz, Yangqing Jia, Lukasz Kaiser, Manjunath Kudlur, Josh Levenberg, Dan Mané, Mike Schuster, Rajat Monga, Sherry Moore, Derek Murray, Chris Olah, Jonathon Shlens, Benoit Steiner, Ilya Sutskever, Kunal Talwar, Paul Tucker, Vincent Vanhoucke, Vijay Vasudevan, Fernanda Viégas, Oriol Vinyals, Pete Warden, Martin Wattenberg, Martin Wicke, Yuan Yu, and Xiaoqiang Zheng. TensorFlow: Large-scale machine learning on heterogeneous systems, 2015. Software available from [tensorflow.org](https://tensorflow.org/).\n",
        "* Chollet, F. (2016, May 14). Building Autoencoders in Keras. Retrieved March 19, 2019, from https://blog.keras.io/building-autoencoders-in-keras.html\n",
        "* Goodfellow, I., Bengio, Y., & Courville, A. (2016). Deep learning. MIT press."
      ]
    }
  ]
}
