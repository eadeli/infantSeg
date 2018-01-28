# infantSeg

This is the project for multi-modality infant brain segmentation at the isointense stage. We develop deep learning based methods to deal with the segmentation task.
More will be uploaded after the journal paper is accepted.

Basic steps:

1. crop patches from medical image data with readMedImg4CaffeCropNie4SingleS.py, I use SimpleITK as readers.

2. prepare the network: define infant_train_test.prototxt, and decide infant_solver.prototxt. And if your network is convolution-free (which I mean is MLP), you can refer to my another proj: https://github.com/ginobilinie/psychoTraitPrediction 

3. train the network with caffe (actually, I also suggestion you try Keras, it is really the simplest deep learning tool I have ever seen, if you are a beginner, Keras may be your best choice).

4. evaluate the caffe model with evalCaffeModel4ImgNie.py, I store them in nifti format.

<B>If you think it is useful for you, please star it. HAHA. And you can cite this paper:</br>
"Nie, Dong, et al. "Fully convolutional networks for multi-modality isointense infant brain image segmentation." Biomedical Imaging (ISBI), 2016 IEEE 13th International Symposium on. IEEE, 2016."</br>
"Dong Nie, Li Wang, Ehsan Adeli, Cuijing Lao, Weili Lin, Dinggang Shen. 3D Fully Convolutional Networks for Multi-Modal Isointense Infant Brain Image Segmentation, IEEE Transactions on Cybernetics, 2018."
</B>


If you want to do projects with caffe in the server, please first refer to http://on-demand.gputechconf.com/gtc/2015/webinar/deep-learning-course/getting-started-with-caffe.pdf (if you are familiar with caffe, please ignore it). And I will introduce detailed steps for developing deep learning applications in medical image analysis with our server based caffe in the following steps.
.........................................................................................................................................................................................................................................................

<B>Here is a list of what the network related files mean, and for the below thre prototxt, I'd like to share some basic experience about how to write them and where we need be careful:</B>

a. infant_train_test.prototxt: define the network architecture</br>
The architecture actually is not hard to define, </br>
number of total layers: the number of layers could be easily initially decided, you can just make the receptive field for the last layer as large as the input. Of course, you should search what receptive field is, and I can give you a suggestion one to read: https://www.quora.com/What-is-a-receptive-field-in-a-convolutional-neural-network.</br>
convolution paramter settings: just see my example (infant_train_test.prototxt), and you can improve it based on this, for example, you can use other initialization methods (I like to use Xavier, you can use others) </br>
activation function: ReLU</br>
fully connected layer setting: refer to prototxt in https://github.com/ginobilinie/psychoTraitPrediction . </br>
....

b. infant_solver.prototxt: define the network hyperparameters</br>
The most important parameters you should take care is learning rate (lr), and learning rate decay strategy (learning_policy: fixed or step or inv, I have examples in this prototxt).  stepsize( this is necessary when you set the learning policy as step, which means it will decrease gamma times when it reaches every stepsize steps).</br>
momutum: you can set by 0.9 as default.</br>
maxIter should be the maximum iterations for the network to train, usually you can set two times as large as you dataset.</br>
And if you want to use other optimization instead of SGD, you have to set 'type', for example, type: "Adam".</br>

c. infant_deploy.prototxt: the deploy prototxt when you want to evaluate your trained caffe model.</br>
You only have to make two changes based on the train_test.prototxt: 
input (replace the original input (e.g., HDF5) with the dimension described format which I have a example in infant_deploy.prototxt.</br>
and also, you have to replace to output softmaxWithLoss with softmax layer.</br>

d. infant_train_test_finetune.prototxt: finetune prototxt when you want to use previous model the initialize the training of a new model.</br>
Notice that we should keep almost everything (instead of learning rate) of the layers you like to initialize the same between the old trained model and the new model, and then you can add new layers in the new model. compare the infant_train_test_finetune.prototxt and infant_train_test.prototxt, you will know how it works.

e. infant_train_test_point.prototxt: define the network architecture</br>
The architecture is for CNN, and it is not hard to define, </br>
number of total layers: the number of layers could be easily initially decided, you can just make the receptive field for the last layer as large as the input. Of course, you should search what receptive field is, and I can give you a suggestion one to read: https://www.quora.com/What-is-a-receptive-field-in-a-convolutional-neural-network.</br>
convolution paramter settings: just see my example (infant_train_test.prototxt), and you can improve it based on this, for example, you can use other initialization methods (I like to use Xavier, you can use others) </br>
activation function: ReLU</br>

f. infant_train_test_unet.prototxt: define a unet-like architecture</br>

g. infant_train_test_singleModality.prototxt: a FCN/unet architecture, but it is single modality</br>

h. infant_train_test_bn.prototxt: a FCN architecture with Bacth Normalization

i: infant_train_test_unet_bn.prototxt: a Unet architecture with Batch Normalization. <B> When doing whole image voxelwise segmentation, I suggest using this one. This is a prototxt achieving good performance and I designed in late 2015 (or early 2016).</B>

j: infant_train_test_23d_resnet_fcn.prototxt: a basic FCN+residual unit

<B>How to train/resume/finetune: (Note, to run it, you have to install caffe which support 3D operations)</B> 

a. train_infant.sh: shell code to train the model

b. resume_infant.sh: resume a previous trained model</br>
If your training is broken by something else, then you should not train from scratch, instead, you can resume the training.

c. finetune_infant.sh: finetune a previous trained model</br>
If you can have more dataset, and then you'd like to finetune the previous trained model, and then you can finetune the trained model. </br>
If you like to train a new model, and you like to fix the first several layers's weights, just train some layers or some new layers, you can also use finetune. In this case, I have edited a infant_train_test_finetune.prototxt, in which the learning rate of the fixing layers is set to 0 (then you can fix these layers' weights, and then finetue it. 
Just use finetune_infant.sh to run it. 

d. If you would like to save the output of screen during training or testing, you can do like this: </br>
nohup bash train_infant.sh >logTrain.txt 2>&1 & </br>

e. trainCaffe.py: You can also <B>train the caffe models with python codes</B>. In this copy of codes, you can also display the network and network parameters.</br>

...........................................................................................................................................................................................................................................................

<B>Here are some codes you may want to use: If you have no IDE for python, I suggestion you use Eclipse+pyDev as editor and compiler for python codes:</B>

a. evalCaffeModels4MedImg.py: this is the code to evaluate a whole 3d image on the trained DL models for the patients. 

b. readMedImg4CaffeCropNie.py: this is the code to extract 3d patches from a medical image (mri/ct and so on) for patients. </br>
Note, I am not that sure if this contains error or not (I wrote too many versions, I donot remember which version it is), if it contains, please let me know by QQ. 

c. readMedImg4CaffeCropNie4SingleS.py: implement the crop patches function. </br>
<B>And this one is much better than readMedImg4CaffeCropNie.py, so I suggest you use this one.</B>

d. evalCaffeModel4ImgNie.py: this is the code to evaluate a whole 3d image on the trained DL models for the patients (<B>suggested using this one, it's good for classification task, but the overlapping part is implemented via averaging, if you like majority voting, please use evalCaffeModel4ImgNieSingSbyMV.py</B>).</br>
And if you want to read the intermedia layer's output, you can specify it by "temppremat = mynet.blobs['layername'].data[0]". Actually, the size of patch is not limited to the size during training, you can make a bigger patch size (than patch size at training stage).

e. transData.py: this is written to transpose (permute in matlab) the dimension order of the h5 (hdf5 in matlab). </br>
Actually, you can also check the hdf5 format data if the label and feature match or not...

f. transImage.py: this is written to transpose (permute in matlab) the dimension order of a medical image.

g. convertMat2MedFormat.py: convert mat files (matlab) to medical image format, such as nifti, hdr and so on

h. checkHDF5.py: check the hdf5 files you generated to find if there are something wrong (e.g., matched or not)

i. imgUtils.py: compute dice/psnr for medical images

j. evalCaffeModel4WholeImgNieSingleS.py Here I take whole image as input
Note, in evalCaffeModel4ImgNie.py, I take a single patch as input, as there are so many patches, it is a little slow, so I now utilize full image as input, but you have to change the deploy prototxt accordingly (just adjust the input dimension). <B>Here, we should keep the order for dimension of the data same with the order of training data (patch).</B>

k: avgMedImg.py: average the medical image predictions, you can use average or majority voting to generate better predictions.

l: readSomeTypeFiles.py: read some specific type of images in a certain directory.

m: evalCaffeModel4ImgNieSingSbyMV.py: this is the code to evaluate a whole 3d image on the trained DL models for the patients (<B>this is for classification task, but I suggest you use evalCaffeSegModel4MedImgbyMV.py when it is classification/segmentation task. Note the overlapping part is implemented via majority voting, if you like averaging, please use evalCaffeModel4ImgNie.py</B>).</br>

n: convertMed2Slice.py: convert nii.gz format to slices

o: evalCaffeSegModel4MedImgbyMV.py: this is the code to evaluate a whole 3d image <B>segmentation</B> on the trained DL models for the patients (<B>suggested using this one, it's good for classification task, Note the overlapping part is implemented via majority voting, if you like averaging, please use evalCaffeSegModel4ImgNie.py</B>).</br>

p: evalCaffeSegModel4ImgNie.py: this is the code to evaluate a whole 3d image <B>segmentation</B> on the trained DL models for the patients (<B>suggested using this one, it's good for classification task, but the overlapping part is implemented via averaging, if you like majority voting, please use evalCaffeSegModel4MedImgbyMV.py</B>).</br>
And if you want to read the intermedia layer's output, you can specify it by "temppremat = mynet.blobs['layername'].data[0]". Actually, the size of patch is not limited to the size during training, you can make a bigger patch size (than patch size at training stage).

q: sepFiles.py: preprocess the files. make directories, cp specific files to each of the filespr
renameFiles.py: preprocess the files: rename files

r: softmax_lossweight_layer.cpp: the weight loss cross entropy loss (can work for 3d, I think it works if wejust merge it into the latest caffe); softmax_lossweight_layer.hpp

s: extractPatch4SegMultiModalMedImg.py: implement the extracting patches function for multi-modality data. </br>
<B>And this one is much better than readMedImg4CaffeCropNie.py, so I suggest you use this one if it is multi-modality data.</B>


