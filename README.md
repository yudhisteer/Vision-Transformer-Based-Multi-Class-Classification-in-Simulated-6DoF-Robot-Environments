# Vision Transformer-Based Multi-Class Classification in Simulated 6DoF Robot Environment

## Problem Statement

## Dataset

## Abstract

## Plan of Action
1. [Generating Synthetic Dataset with Unity](#syntehtic)
2. [A Gentle Introduction to Attention](#attention)
3. [Vision Transformer: How much is an image worth?](#vision)
4. [Coding Transformers for Image Recognition: From Pixels to Predictions](#transformer)
5. [Simulating Palletizing with Transformers](#simulation)

-----------------
<a name="syntehtic"></a>
## 1. Generating Synthetic Dataset with Unity

### 1.1 Object Classes
As the main goal of the project is an image classification task, we have chosen three objects that have distinct 3D structures and features. The 3D models are a [cardboard box](https://sketchfab.com/3d-models/small-cardboard-box-closed-9f0345c78b7b4761b9cdec5393474bd1), a [plate](https://sketchfab.com/3d-models/lunch-plate-school-project-eef24ebe601c4e2f99da3108ddc3b09b), and a [vase](https://sketchfab.com/3d-models/ancient-vase-dce37778ec964299bba5aeca736bf70e) and they were downloaded from [Sketchfab](https://sketchfab.com/).

<p align="center">
  <img src="https://github.com/yudhisteer/Vision-Transformer-Based-Multi-Class-Classification-in-Simulated-6DoF-Robot-Environments/assets/59663734/2205306a-c5cb-4178-897f-35f0625456b4" width="30%" />
  <img src="https://github.com/yudhisteer/Vision-Transformer-Based-Multi-Class-Classification-in-Simulated-6DoF-Robot-Environments/assets/59663734/c730440f-ce4b-4079-9ea7-cb9b2b2bfc2f" width="30%" />
  <img src="https://github.com/yudhisteer/Vision-Transformer-Based-Multi-Class-Classification-in-Simulated-6DoF-Robot-Environments/assets/59663734/0a39f30b-056d-4589-9fd3-b4741f6d15b5" width="30%" />
</p>

After downloading from Sketchfab, we import them on **Unity (2022.3.9.f1 version)**. We create a scene with a **camera**, a **light**, and a **plane**. We place our 3D object on the plane at ```(0,0,0)``` position and place the camera above the 3D object. We ensure in the game scene the object is not too far or too close to the camera. We also tilt the camera slightly, around 70 degrees, so that we can have a clear 3D view of the object. Below is an example of the setup:

<p align="center">
  <img src="https://github.com/yudhisteer/Vision-Transformer-Based-Multi-Class-Classification-in-Simulated-6DoF-Robot-Environments/assets/59663734/90051c79-13f2-4ffc-8fb3-4dc071bbcf3b" width="90%" />
</p>


Note that the game scene represents the output image. We choose a resolution of ```400x400```. What we have demonstrated is how we can generate a synthetic image from 3D models using Unity. However, it's worth noting that we have only created a single image with a fixed object size and color, a uniform plane color, a specific camera position, and a singular lighting configuration. To build a robust classification model, we must acquire more data. Unfortunately, manually altering these parameters for each instance is a time-consuming and labor-intensive task. We must automate this process.

In order to generate large-scale synthetic datasets, we will use the [Perception Package](https://docs.unity3d.com/Packages/com.unity.perception@1.0/manual/index.html). More specifically, we will use the **Randomization tool** that allows us to integrate domain randomization principles into our simulation.

### 1.2 Camera Randomizer
We start by creating a script that will randomly position and rotate our camera in the scene. We will randomly select a value for the distance (about the y-axis) and the elevation (about the z-axis). We then calculate the camera's new position in a spherical coordinate system using the elevation and distance values. We update the camera's rotation and position based on the sampled values. We set the camera distance to be between ```5``` and ```8``` units whereas the rotation about the x-axis to be between ```15``` and ```95``` degrees. 

```c#
    // Sample a random elevation and distance.
    float elevation = cameraRotateX.Sample();
    float distance = cameraDistance.Sample();

    // Calculate the camera's new position in a spherical coordinate system.
    float z = -distance * Mathf.Cos(elevation * Mathf.PI / 180);
    float y = distance * Mathf.Sin(elevation * Mathf.PI / 180);

    // Update the camera's rotation and position based on the sampled values.
    myCamera.transform.rotation = Quaternion.Euler(elevation, 0, 0);
    myCamera.transform.position = new Vector3(0, y, z);
```

### 1.3 Plane Color Randomizer
Next, we want our plane to change color for each frame. We create a new script that will randomly sample an  RGB color value. We then apply that color to the plane. Note that our script will iterate for each frame depending on the number of iterations we set. the colors could be any values between ```0``` and ```255```.

```c#
    // Sample a random RGB color value.
    Color randomizedColor = colorRGB.Sample();

    // Apply the randomized color to the selected material.
    selectedMat.color = randomizedColor;
```

### 1.4 Light Randomizer
Similarly, we want the lighting in our sample to be different for each frame. We randomly sample rotation values in the x,y, and z coordinates and apply them to the lighting object. We set the light intensity to change between ```0.5``` and ```3```. The rotation about the x-axis should be between ```15``` and ```90``` degrees, about the y-axis should be between ```-180``` and ```180``` and about the z-axis should be between ```0``` and ```1``` degrees. 

```c#
    // Randomize the rotation of the light using the sampled values.
    tagLight.transform.eulerAngles = new Vector3(lightRotateX.Sample(), lightRotateY.Sample(), lightRotateZ.Sample());
```

### 1.5 Object Placement Randomizer
Furthermore, we do not want our object to be fixed in the center of the plane. We want the object to have a new **position**, **rotation**, and **scale** for each frame. We thus sample values for these three properties and apply them to the instantiated object. We set the scale of the object to vary between ```0.8``` and ```1.2```, the placement rotation to be between ```-180``` and ```180``` about the y-axis only, and the placement location to vary between ```-1.8``` and ```+1.8```.

```c#
    // Set the position, rotation, and scale of the instantiated object based on sampled values.
    currentInstance.transform.position = placementLocation.Sample();
    currentInstance.transform.eulerAngles = placementRotation.Sample();
    currentInstance.transform.localScale = Vector3.one * objScale.Sample();
```


### 1.6 Object Color Randomizer
Lastly, we want our object to be of different colors for each iteration. Similarly, to the plane color randomizer, we sample random RGB values and apply them to our object. We need to make sure we are using the color material associated with our prefab 3D object.

```c#
    // Sample a random RGB color value.
    Color randomizedColor = colorRGB.Sample();

    // Apply the randomized color to the selected material.
    selectedMat.color = randomizedColor;
```



### 1.7 Simulation
We then run the simulation for 150 iterations. Note that we create a new scene for each object. We tried to spawn the 3D objects randomly one at a time in one scene only however, all images will be stored in one folder and it defeats the purpose of building the image classification if we manually need to filter the images at this stage. It can be a good exercise to test our model with this random spawned 3D model approach in the end though.  Below is a step-by-step output of the result of the simulation:

<div style="text-align: center;">
  <video src="https://github.com/yudhisteer/Vision-Transformer-Based-Multi-Class-Classification-in-Simulated-6DoF-Robot-Environments/assets/59663734/0ada2f9b-20a7-4069-89c4-30bceff2b2aa" controls="controls" style="max-width: 730px;">
  </video>
</div>

We opted for an iteration count of ```150```. In the image classification format, images are organized into distinct directories labeled with specific class names. For instance, all images of plates are stored in the "plate/" directory. This format is widely used in various image classification benchmarks, such as ImageNet. Below, you can find an example of this storage format, with arbitrary image numbers.

```text
cardboard_plate_vase/ <- overall dataset folder
    train/ <- training images
        cardboard/ <- class name as folder name
            image01.jpeg
            image02.jpeg
            ...
        plate/
            image24.jpeg
            image25.jpeg
            ...
        vase/
            image37.jpeg
            ...
    test/ <- testing images
        cardboard/
            image101.jpeg
            image102.jpeg
            ...
        plate/
            image154.jpeg
            image155.jpeg
            ...
        vase/
            image167.jpeg
            ...
```

Below is the output of the images for the plate object class:

<p align="center">
  <img src="https://github.com/yudhisteer/Vision-Transformer-Based-Multi-Class-Classification-in-Simulated-6DoF-Robot-Environments/assets/59663734/7ce21473-5ed3-40bd-8407-6761ad2c82b2" />
</p>

-----------------
<a name="attention"></a>
## 2. A Gentle Introduction to Attention

-----------------
<a name="vision"></a>
## 3. Vision Transformer: How much is an image worth?
Most of the explanation found below comes from the Vision Transformer paper itself: [An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale
](https://arxiv.org/abs/2010.11929)

### 3.1 Overview
Before we dive into Transformer, let's do a quick overview of how CNN works first. 

1. A Convolutional Neural Network (CNN) uses **kernels** to gather **local** information in each layer.
2. This information is subsequently passed to the next layer, which further aggregates local data with an enlarged **receptive field**.
3. The latter occurs because the final layer considers information that has already been **aggregated** by the initial layer.
4. Initially, CNNs focus on local areas to capture **simple patterns** such as **edges**, and as they progress through the layers, their receptive fields become more **global**. This enables them to capture more complex patterns and semantics.

The author of the ViT paper claims that they did not rely on attention in conjunction with CNN but a solely Transformer architecture was used directly to a sequence of patches of images that outperformed SOTA image classification models. Self-attention-based architectures - Transformers - have been the SOTA method for many NLP tasks. However, CNN remained dominant in computer vision. In order to apply, the Transformer network to a computer vision task, they had to transform their input data (images) the same way the input (tokens) was for a Transformer network in NLP. They split the image into patches of ```16x16``` pixels. These patches are then flattened by a linear transformation matrix to become vectors. The vector of each patch gets a positional embedding and a Transformer is used to predict the class of the image. 

However, the ViT model underperformed when trained on mid-size datasets because of a lack of "**inductive bias**" (I later explain this term). But when trained on **larger datasets** (```14M-300M```) images, they outperformed SOTA image recognition models and concluded that "_large  scale training trumps inductive bias_". 

In summary:

- The Transformer requires **abundant data** because it has the freedom to look **everywhere** at the image from the start. That is, it is unfocused at the start and needs a huge amount of data to learn **what** and **where** to focus to make the right predictions.
- The Transformer can find **novel** ways to look at the data because it isn't guided on how to do so.
- On the other hand, CNN is focused in the beginning by the convolutions towards a **local view**. The given focus patterns can be a limitation but we spare a lot of training data because we do not have to teach the model how to focus but only **where** to focus. 



### 3.2 The Architecture



<p align="center">
  <img src="https://github.com/yudhisteer/Vision-Transformer-Based-Multi-Class-Classification-in-Simulated-6DoF-Robot-Environments/assets/59663734/51e48470-3a5a-4d2f-8b4c-91c53f1a607a" />
</p>

### 3.3 Equation 1
The author explains that the original Transformer for the NLP task takes a sequence of token embeddings as input. Hence, we need to transform our images into the same. 

<p align="center">
  <img src="https://github.com/yudhisteer/Vision-Transformer-Based-Multi-Class-Classification-in-Simulated-6DoF-Robot-Environments/assets/59663734/3d13cde2-d641-4d27-a2e7-aaf3d80c5a27" />
</p>

where

<p align="center">
  <img src="https://github.com/yudhisteer/Vision-Transformer-Based-Multi-Class-Classification-in-Simulated-6DoF-Robot-Environments/assets/59663734/71ffefd2-a534-41a6-b165-99a5c36c786a" />
</p>

and

<p align="center">
  <img src="https://github.com/yudhisteer/Vision-Transformer-Based-Multi-Class-Classification-in-Simulated-6DoF-Robot-Environments/assets/59663734/86bd48b5-aca0-4844-8f34-7ecfc8052ec3" />
</p>


The representation of our input is to split an image into fixed-size patches and then return each patch of the image into a **learnable embedding**. Our original image is of size ```224 x 224 x 3 -> [Height x Width x Color Channel]```. We choose a patch size of ```16 x 16```, therefore the resulting number of patches is:

<p align="center">
  <img src="https://github.com/yudhisteer/Vision-Transformer-Based-Multi-Class-Classification-in-Simulated-6DoF-Robot-Environments/assets/59663734/f5622088-ad7e-4fe3-8d7b-0fb9d208eed8" />
</p>

Hence, the output shape of a single 2D image flattened into patches will be of size: ```196 x 768 -> [N x (P^2 ⋅ C)]```. Note that ```196``` is the input sequence length for the transformer and ```768``` is the embedding dimension.


<p align="center">
  <img src="https://github.com/yudhisteer/Vision-Transformer-Based-Multi-Class-Classification-in-Simulated-6DoF-Robot-Environments/assets/59663734/39fa623a-d35a-4272-9735-0f0f4a8734f0" width="90%"/>
</p>



### 3.4 Equation 2

### 3.5 Equation 3

### 3.6 Equation 4



-----------------
<a name="transformer"></a>
## 4. Coding Transformers for Image Recognition: From Pixels to Predictions

### 4.1 Equation 1

We will start by coding equation 1 which is to first transform our input image into patches. 



<p align="center">
  <img src="https://github.com/yudhisteer/Vision-Transformer-Based-Multi-Class-Classification-in-Simulated-6DoF-Robot-Environments/assets/59663734/4e5658d7-366b-48b8-a982-4859296d0333" />
</p>


**Pseudocode:**

    x_input = [class_token, patch_1, patch_2, ..., patch_N] + [class_token_pos, patch_1_pos, patch_2_pos, ..., patch_N_pos]

#### 4.1.1 Patching

There are 2 ways to turn our image into patches:

**1) Using raw image patches**

In the figure below, we looped through the different height and width dimensions of a single image and plotted individual patches. However, this method can be computationally expensive and time-consuming. It took ```16.780``` seconds to output the image below.

<p align="center">
  <img src="https://github.com/yudhisteer/Vision-Transformer-Based-Multi-Class-Classification-in-Simulated-6DoF-Robot-Environments/assets/59663734/48e040df-339d-42ec-b032-b6bf87a4cf1e" width="50%"/>
</p>

**2) Using feature maps**

The author of the ViT paper proposed a **hybrid model** in which we can use the **feature maps** of a CNN. The patches can have a spatial dimension of ```1x1```, that is the input sequence is created by flattening the spatial dimensions of the feature map and then projecting it into the Transformer dimension.


#### 4.1.2 Class Token

#### 4.1.3 Position Embedding

#### 4.1.4 Patch + Position Embedding


### 4.2 Equation 2


### 4.3 Equation 3

### 4.4 Equation 4


-----------------
<a name="simulation"></a>
## 5. Simulating Palletizing with Transformers









----------
## Conclusion

## References
1. https://arxiv.org/abs/2010.11929
2. https://www.youtube.com/watch?v=TrdevFK_am4&ab_channel=YannicKilcher
3. https://www.youtube.com/watch?v=j3VNqtJUoz0&ab_channel=DeepFindr
4. https://www.youtube.com/watch?v=DVoHvmww2lQ&list=PLpZBeKTZRGPMddKHcsJAOIghV8MwzwQV6&index=1&ab_channel=AICoffeeBreakwithLetitia
5. https://www.youtube.com/watch?v=j6kuz_NqkG0&ab_channel=AleksaGordi%C4%87-TheAIEpiphany
6. https://www.youtube.com/watch?v=DVoHvmww2lQ&list=PLpZBeKTZRGPMddKHcsJAOIghV8MwzwQV6&ab_channel=AICoffeeBreakwithLetitia
