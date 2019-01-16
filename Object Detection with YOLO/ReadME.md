## Object Detection using Google AI Open Images

![](https://github.com/sagar-chadha/Data-Science-Projects/blob/master/Repository%20Files/Object%20Detection%20with%20YOLO/obj_det.png)

My team of budding data scientists and I decided to try our hands at the [Google AI Open Image challenge](https://www.kaggle.com/c/google-ai-open-images-object-detection-track) hosted on Kaggle. We thought of this as the perfect opportunity to get our hands dirty with neural networks and convolutions, and potentially impress our professors and classmates. 

### The Task
This challenge provided us with 1.7 million images with 12 million bounding box annotations (their X and Y coordinates relative to the image) of 500 object classes. You can find the data [here](https://www.figure-eight.com/dataset/open-images-annotated-with-bounding-boxes/). <br>

Being our first foray into computer vision coupled with the fact that we dont have humungous computational resources, we decided to use the [Google Cloud Platform](https://cloud.google.com/) to train a small number of frequently appearing classes.

### The approach
A look at the training images revealed that certain objects had more of a presence than others in terms of how many times they appeared. The chart below shows the distribution of some of the classes.
![](https://github.com/sagar-chadha/Data-Science-Projects/blob/master/Repository%20Files/Object%20Detection%20with%20YOLO/freq_chart.png)

* We chose the aforementioned **43 object classes and a subset of ~ 300K images** with these objects. We had atleast 400 images for each object class in the training data.
* We decided to use a pretrained **YOLO v2.0 algorithm** because of its speed, computational power and the abundance of online articles that could guide us through the process. We then decided to re-train the last layer of the model using our images so that it could predict new classes of objects (Transfer Learning)

### Concepts Used
* Convolutional Neural Networks (Take this [course](https://www.coursera.org/learn/convolutional-neural-networks) for an excellent introduction to CNNs)
* Transfer Learning
* YOLO model

### Results
We trained the last layer of the YOLO model for 20-30 epochs to get it to recognize the new objects that we were training with with satisfactory confidence levels. An example of an output that we got from the CNN is shown below - 
![](https://github.com/sagar-chadha/Data-Science-Projects/blob/master/Repository%20Files/Object%20Detection%20with%20YOLO/result.png)

We were surprised how well the model recognized man and woman even if they were facing away. This is probably because man and woman had a lot of appearances in photos (as the class frequency chart above shows).

For a detailed account of our work, you can read our team's article on this project on [Medium](https://towardsdatascience.com/object-detection-using-google-ai-open-images-4c908cad4a54).

### Repository Structure
`Object_Detection.ipynb` file is the code that we used to perform this computer vision task. Since we couldn't run it on our local machines, outputs would be missing in most places.
