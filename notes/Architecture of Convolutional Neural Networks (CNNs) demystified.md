### [Architecture of Convolutional Neural Networks (CNNs) demystified](https://www.analyticsvidhya.com/blog/2017/06/architecture-of-convolutional-neural-networks-simplified-demystified/)
- Fully connected network unable to preserve the spatial arrangement of the image. Flattening the image destroys its arrangement completely. **-->** we need to devise a way to send images to a network without flattening them and retaining its spatial arrangement.
- The weight matrix behaves like a filter in an image extracting particular information from the original image matrix.
![](https://s3-ap-south-1.amazonaws.com/av-blog-media/wp-content/uploads/2017/06/28132834/convimages.png)
- The convolution and pooling layers would only be able to extract features and reduce the number of parameters from the  original images.
- To generate the final output we need to apply a fully connected layer to generate an output equal to the number of classes we need.
