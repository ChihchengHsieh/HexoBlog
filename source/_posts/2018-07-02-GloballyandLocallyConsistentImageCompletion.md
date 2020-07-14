---
title: Globally and Locally Consistent Image Completion
date: 2018-07-02
categories:
- DL
tags:
- Globally and Locally Consistent Image Completion
photos:
- https://github.com/mike820808/tf_Globally_and_Locally_Consistent_Image_Completion/blob/master/Photo/PaperStructure.png?raw=true
---

<!--more-->

[[Paper](http://hi.cs.waseda.ac.jp/~iizuka/projects/completion/en/)][[My minimal pytorch implementation](https://github.com/ChihchengHsieh/Globally_and_Locally_Consistent_Image_Completion)]

# What's different?

* Move the model to Pytorch
* Using High Level API (tf.layers/tf.contrib)
* add batch normalisation after dilated CNN
<img src="https://github.com/mike820808/tf_Globally_and_Locally_Consistent_Image_Completion/blob/master/Photo/BN.png?raw=true" alt="img" width="500" height="150">
* This model run on 64x64 size images
* The structure of model has been changed a liitle bit since the smaller size image
* Add the sigmoid function in the last layer of Completion

# Paper Structure
<img src="https://github.com/mike820808/tf_Globally_and_Locally_Consistent_Image_Completion/blob/master/Photo/PaperC.png?raw=true" alt="img" width="325" height="300">
<img src="https://github.com/mike820808/tf_Globally_and_Locally_Consistent_Image_Completion/blob/master/Photo/PaperD.png?raw=true" alt="img" width="350" height="300">

# Paper optimization
<img src="https://github.com/mike820808/tf_Globally_and_Locally_Consistent_Image_Completion/blob/master/Photo/Optim.png?raw=true" alt="img" width="350" height="300">

# The structure for this model
![](https://github.com/mike820808/tf_Globally_and_Locally_Consistent_Image_Completion/blob/master/Photo/Sructure.png?raw=true)

# The result after only 20 steps (5 minutes) 

<- Yeah .. I mean 5 min on CPU...

The generator has started to do something we want. The reason may be the reconstructed loss, which enables us to train it as a decoder.

![](https://github.com/mike820808/tf_Globally_and_Locally_Consistent_Image_Completion/blob/master/Photo/20%20epoch%20result.png?raw=true)

And, the paper they trained

<img src="https://github.com/mike820808/Globally_and_Locally_Consistent_Image_Completion/blob/master/Photo/Training_time.png?raw=true" alt="img" width="500" height="60">


# Still wondering

Which way of initialization will be better for this model?
