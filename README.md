# self_driving_gopigo



# Self Driving GoPiGo

![thumbnail](/images/thumbnail.jpg) 

First of all to get your attention right away:
[here is the video of the car driving.](https://youtu.be/DT_9L6zDL5M)
Now you want to know how it works, don't you? Don't worry I am going to explain now. 

As you can see above, the goal is that the car won't drive out of the region that is surrounded by the black border.

The hardware of the car is the GoPiGo2 kit from Dexter Industries. You only have to assemble it by yourself. This is quite simple, it's just as easy as connecting some lego bricks.
To control the car I just call the python API from the GoPiGo. As I will mention in the next right next, the neural network is inspired and taken from a book with some adaptions. Besides that, I developed and wrote everything by myself.
That includes:

* collecting and preparing the images as a dataset for the convolutional neural network (CNN),
* training the CNN in the could with Floydhub,
* saving the tensorflow model and using it on the Raspberry Pi, 
* the UI that controls the car and that can start the autopilot,
* and the algorithm that chooses the next move for the car. 

The heart of the algorithm is a CNN that is trained to decide whether a piece of the floor is save for driving or if it is a border and the car needs so top or change its direction. For that, I collected images and created a dataset. After this a trained the network with tensorflow. 
The architecture of the net is based on the network for the MNIST dataset form the book “Fundamentals of Deep Learning: Designing Next-Generation Machine Intelligence Algorithms” from Nikhil Buduma. 
It is a quite basic network with two layers of convolution and max-pooling and two fully connected layers.
Nonetheless, it gives enough performance to provide a quite stable autopilot for the car. 



The algorithm is divided into two parts: 

1. The image is classified and a so-called autopilotmatrix $$A$$ is generated.

2. With a given $$A$$ a driving instruction for the car is determined. 


The first point works the following way. The camera of the Raspberry Pi takes a picture which is divided into a grid of size $$28 \times 28$$ pixel. As you can see in the pictures, only the cells corresponding to the autopilotmatrix $$A$$ are of interest. 
The CNN takes a cell as input and predicts if it is save for driving or it is part of the border. If the CNN classifies a cell of the floor as save, then the value of the entry of the matrix is 1, otherwise, it is 0.  


The second part is now described. You can look at the autopilot as a mapping from the set of autopilotmatrices $$\mathcal{A}$$ to the set of possibles moves $$\mathcal{M} := \{f(v), b\} $$ of the car. 
$$f(v)$$ is moving forward with a velocity $$v$$ and $$b$$ is moving backward and take a new direction which is random. 


$$ f \colon \mathcal{A} \to \mathcal{M}$$


This mapping works the following way. Let $$T$$ be a $$ 4 \times 3$$ Matrix and a $$\mathbf{v}$$ be a $$4 \times 1$$ vector. The names indicate the meaning, $$T$$ stand for turn and $$\mathbf{v}$$ stands for velocity. 
So the autopilotmatrix is 

 $$A = \begin{pmatrix}
 0 & v_1 & 0 \\
 0 & v_2 & 0 \\
 0 & v_3 & 0 \\ 
 0 & v_4 & 0 \\
 a_{11} &  a_{12} & a_{13}  \\
 a_{21} &  a_{22} & a_{23}  \\ 
 a_{31} &  a_{32} & a_{33}  \\
 a_{41} &  a_{42} & a_{43} 
 \end{pmatrix}$$
 
If $$\Vert A \Vert_1 = 0$$, that is if and only if every entry of $$A$$ is $$0$$, the car should drive just straight ahead. The velocity of this movement is determined by $$\mathbf{v}$$. If there is no border in sight, that is when $$ \Vert \mathbf{v} \Vert_1 = 4$$, the velocity is set to maximum. If not, the velocity is reduced to a certain value.
With this, I try to avoid that the car is driving to fast towards the border and probably drives over the border without recognizing it.  

If $$\Vert A \Vert_1 < 2$$ the car is to close to the border and so it has to stop moving forward. It will then drive back and after that, it will turn randomly left or right. After that, the algorithm starts again and the car drives straight ahead. 


Here is the UI:


![UI](/images/1.png)


As you can see it's called "Car v0.1 on Windows". I wrote the software so I can start it with no dependencies of the operating system. When I'm on windows I'm using a fake camera to simulate the camera of the raspberry pi.
These are just some pictures in a folder on my computer. 

Here is an image where the car should stop.
![all_black](/images/2.png)

Here is another image. 
![half_black](/images/3.png)


Of course, the car drives not that fast and the approach its pretty simple. But the car is able to drive in the marked area without leaving it. From this, you could develop more features and add more machine learning algorithms. For example, you could place a traffic sign somewhere that the car should find and drive towards. Hopefully, I will find some time in the future to continue working on this project. 