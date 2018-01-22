# py-rfcn Example
R-FCN: Object Detection via Region-based Fully Convolutional Networks

This example follows hhttps://github.com/YuwenXiong/py-R-FCN and provides examples to run py-RFCN training based on UAI Environments(including using UAI gpu docker image and using UAI Train platform)

We present how to run end2end training based on pretrained resnet model in this example.

## Setup
You should follow https://github.com/rbgirshick/py-faster-rcnn/README.md to download VOCdevkit2007 and resnet101 model. Or you can follow https://github.com/YuwenXiong/py-R-FCN to download both VOCdevkit2007/2012 dataset and ResNet50 model

UCloud provides basic py-R-FCN docker images through uhub.ucloud.cn/uaishare/gpu\_uaitrain\_ubuntu-14.04\_python-2.7.6\_caffe-py-rfcn:v1.0, you can download this docker after register a UCloud account.

The image includes:

  - cudnn6.0+cuda8.0
  - YuwenXiong/py-R-FCN
  - Microsoft caffe
  - uai sdk

py-faster-rcnn location in the image: /root/caffe-py-rfcn/

caffe location in the image: /root/caffe-py-rfcn/caffe/

## Intro
The entry point of the training progress is tools/train_net.py. We have modified it to run on UAI platforms. All codes are in examples/caffe/train/rfcn/code/tools are directly copied from 'caffe-py-rfcn/tools' with appropreate modification.

### Code modifications
We need to do several modifications to the original code from YuwenXiong's to make RFCN train tasks running on UAI Platform. These modificiations have already done and packed/compiled into the docker imange we provided.

  - lib/fast\_rcnn/config.py
  - lib/fast_rcnn/config.py
  - lib/rpn/anchor_target_layer.py

#### Modify lib/fast\_rcnn/config.py
	# L199: Make the default ROOT_DIR to /data/
	#       /data/data/ will be the target for input 
	#       /data/output will be the target for output
	__C.ROOT_DIR = '/data/' 

#### Modify lib/rpn/anchor\_target\_layer.py
	# L85: cast idx to int, as numpy > 1.11 does not support float idx
	fg_inds = npr.choice(fg_inds, size=int(fg_rois_per_this_image), replace=False) 

## UAI Example
### Prepare for packing training image
Suppose we put every thing in /voc_test/

	$ cd /voc_test/
	$ ls
	$ code base_tool.py data output

  - base_tool.py is copied from uaitrain\_tool
  - code dir contains tools/ code
  - data dir contains necessary data for testing training locally (including VOCdevkit2007 and models)
  - output dir is the target for output

### Preparing codes
We need to do several modifications to the original code in tools/ to let the train runnable

  - tools/train\_net.py
  - tools/\_init\_paths.py

We have already modifed the code for you. All files under code/ are ready to go.

#### Modify tools/train\_net.py
	# L11: import os lib

#### Modify tools/\_init\_paths.py
	# L17: add python path /root/caffe-py-rfcn/caffe/
	# and /root/caffe-py-rfcn/lib/
	
	# this_dir = osp.dirname(__file__)                                                                                                                             
	this_dir = '/root/caffe-py-rfcn/'

	# Add caffe to PYTHONPATH                                                                                                                                      
	caffe_path = osp.join(this_dir, 'caffe', 'python')
	add_path(caffe_path)

	# Add lib to PYTHONPATH                                                                                                                                        
	lib_path = osp.join(this_dir, 'lib')
	add_path(lib_path)

### Preparing data
We should put VOCdevkit2007 data into /voc\_test/data/ and put models (same as YuwenXiong/py-R-FCN/models/) into /voc\_test/data. Futher we put resnet101_rfcn_final.caffemodel into /voc\_test/data/models/models/.

	$ ls /voc_test/data
	VOCdevkit2007/
	models/

### Packing the end2end traing image
	$ cd /voc_test/
		
The pack cmd will generate executable docker cmd to run the training

	 sudo nvidia-docker run -it -v /voc_test/data:/data/data -v /voc_test/output:/data/output uhub.ucloud.cn/<YOUR_REGISTRY>/caffe-rfcn:uaitrain /bin/bash -c "cd /data && /usr/bin/python /data/tools/train_net.py --solver=/data/data/models/pascal_voc/ResNet-101/rfcn_end2end/solver.prototxt --weights=/data/data/models/models/resnet101_rfcn_final.caffemodel --cfg experiments/cfgs/faster_rcnn_end2end.yml --num_gpus=1 --work_dir=/data --data_dir=/data/data --output_dir=/data/output --log_dir=/data/output"

### Run on UAI Train Platform
Please see https://docs.ucloud.cn/ai/uai-train/ for more information and contact our sales from ucloud.cn