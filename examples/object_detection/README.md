# Object detection on COCO

This example shows you how to build `train`, `test` and `infer` processes
for object detection problems. In particular, it will show you the format
of the return of `test` and `infer`. For an object detection problem, those
return can be a little bit involved. But to get most out of Alectio's platform,
those returns needs to be correct. 

Since the goal of this tutorial is to show you how to setup a pipeline, 
we are not going to train on the entire COCO dataset. Instead, we will 
only use 100 samples for training set and 100 samples for test set. 
And we will use yolo-v3 as the model.

*** All of the following steps assume that your terminal points to the current directory, i.e. `./examples/object_detection` *** 

### 1. Set up a virtual environment and install Alection SDK
Before getting started, please make sure you completed the [initial installation instructions](../../README.md) to set-up your environment. 

To recap, the steps were setting up a virtual environment and then installing the AlectioSDK in that environment. 

To install the AlectioSDK from within the current directory (`./examples/object_detection`) run:

```
pip install ../../.
```

### 2. Install Requirements

Install the requirements via:
```
pip install -r requirements.txt
```

### 3. Download Pre-Processed Data
Create a data directory in the current directory and download the data.

```
mkdir data
cd data
aws s3 cp s3://alectio-resources/cocosamples . --recursive
cd .. 
```

Please note that to run the above command, you need to first [configure the aws cli](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html) by running
```
aws configure
```
and adding your AWS credentials. For most part the data is preprocessed according to the Darknet convention. The only difference is that we use `xyxy` for ground-truth bounding box. 

### 3. Build Train Process
We will train a [Darknet yolov3](https://pjreddie.com/media/files/papers/YOLOv3.pdf) for
this demo. The model is defined in `model.py` the configuration file that specifies the
architecture of the model is defined in `yolov3.cfg`.

To try out this step, run:

```
python model.py
```

Refer to main [AlectioSDK ReadMe](../../README.md) for general information regarding the 
arguments of this process.

### 4. Build Test Process
The test process tests the model trained in each active learning loop.
In this example, the test process is the `test` function defined 
in `processes.py`. 

To try out this step, first create a log dir via

```
mkdir log
```
and then run

```
python processes.py
```

Refer to main [AlectioSDK Readme](../../README.md) for general information regarding the 
arguments of this process.

#### Return of the Test Process 
You will need to run non-maximum suppression on the predictions on the test images and return 
the final detections along with the ground-truth bounding boxes and objects
on each image. 

The return of the `test` function is a dictionary 
```
    {"predictions": prd, "labels": lbs}
    
```

`prd` is a dictionary whose keys are the indices of the test 
images. The value of `prd[i]` is a dictionary that records the final
detections on test image `i`,

**Keys of `prd[i]`**

boxes (`List[List[float]]`)
>  A list of detected bouding boxes. 
    Each bounding box should be normalized according 
    to the dimension of test image `i` and it 
    should be in `xyxy`-format.
  
scores (`List[float]`)
> A list of objectedness of each detected
   bounding box. Objectness should be in \[0, 1\].

objects (`List[int]`)
> A list of class label of each detected 
    bounding box. 


`lbs` is a dictionary whose keys are the indices of the test images. 
The value `lbs[i]` is a dictionary that records the ground-truth bounding 
boxes and class labels on the image `i`.

**Keys of `lbs[i]`**

boxes (`List[List[float]]`)
> A list of ground-truth bounding boxes on image `i`.
    Each bounding box should normalized according to the dimension
    of test image `i` and it should be in `xyxy`-format.
 
objects (`List[int]`)
> A list of class label of each ground-truth bounding box.

difficulties (`Optional[List[{0,1}]]`)
> A list of difficulties of each ground-truth object. 
   An object is 'difficult' if it is difficult for human to detect. 
   For example, an object can be difficult if it is extremely small. 
   Use 1 for difficult objects and 0 for non-difficult objects.
   Difficult objects will not be accounted when calculating mAP.
   If you skip this field, then all objects are assumed to be non-difficult
  

### 5. Build Infer Process
The infer process is used to apply the model to the unlabeled set to run inference. 
We will use the inferred output to estimate which of those unlabeled data will
be most valuable to your model.

Refer to main [AlectioSDK ReadMe](../../README.md) for general information regarding the 
arguments of this process.

#### Return of the Infer Process
The return of the infer process is a dictionary
```python
{"outputs": outputs}
```

`outputs` is a dictionary whose keys are the indices of the unlabeled
images. The value of `outputs[i]` is a dictionary that records the output of
the model on training image `i`. 

**Keys of `outputs[i]`**
boxes (`List[List[float]]`)
> A list of detected bounding boxes.
    You need to apply non-maximum suppression on all predicted bounding 
    boxes. 
    Each bounding box should be normalized according 
    to the dimension of test image `i` and it 
    should be in `xyxy`-format.
  
scores (`List[float]`)
>  A list of objectedness of each detected
   bounding box. Objectness should be in `[0, 1]`.

pre_softmax (`List[List[float]]`):
> A list of class distribution for each 
    detected bounding boxes without applying softmax.



