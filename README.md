# YoloV5 export to Google Coral Dev Mini board

It is a N step process:

1. Upgrade the board to latest version and all software packages
1. Convert model weights to tflite
2. Run the inference on Coral Dev Mini

## Prerequisites

To make sure you have the right environment please flash the latest version [2020](https://coral.ai/software/#mendel-dev-board-mini) of the Google Coral Dev Mini board by following the steps [here](https://coral.ai/docs/dev-board-mini/reflash/).

ATTENTION! This will erase completely the board and loose any work you might have. Also this will erase network configuration so you need to follow the [initiation procedure](https://coral.ai/docs/dev-board-mini/get-started/) in order to connect to Wifi.

Once the board is ready with the new version, upgrade all the package on the board as described [here](https://coral.ai/docs/dev-board-mini/get-started/#6-update-the-mendel-software), and beware for system date to be correct.

```
sudo apt-get update

sudo apt-get dist-upgrade

sudo reboot now
```

install requirements.txt

install opencv for Coral as described in this link: https://krakensystems.co/blog/2020/doing-machine-vision-on-google-coral  I did the steps in the Quick Solution chapter.

Or take 12 hours to compile it yourself as described in this issue: https://github.com/google-coral/examples-camera/issues/76

https://colab.research.google.com/drive/1BDX8bjOGyxl6xWFh8mXtheAi36DTa98S?usp=sharing

### Convert Model Weights to tflite


Convert

##### And if you want to perform the conversion on your system then follow bellow instructions:

I recommend create a new conda environment for this as we need python==3.7.0 for this: 

```
conda create -n yolov5_env python==3.7.0
conda activate yolov5_env
```

Then run below commands and replace **yolov5s.pt** with your own model path and also change **yolov5s.yaml** accordingly. 

```bash
git clone https://github.com/karanjakhar/yolov5.git
cd yolov5
pip3 install tensorflow==2.3.1
pip install -r requirements.txt
python3 models/tf.py --weight yolov5s.pt --cfg models/yolov5s.yaml --img 416 
```



### Run the inference on Raspberry Pi

Just clone this repository and we are good to go:

```bash
git clone https://github.com/karanjakhar/yolov5-export-to-raspberry-pi.git
cd yolov5-export-to-raspberry-pi
chmod +x setup.sh
./setup.sh
python3 yolov5_tflite_folder_of_images_inference.py --weights yolov5s-fp16.tflite --folder_path images/ 
```

Move your own model tflite file to raspberry pi and use that with above command. 



**yolov5_tflite_inference.py**   this file contains main inference code which you can use with your own project. 

Other files show examples how to use it. I have placed a tflite file and some sample images to run a quick test. 





**To Run Examples:** 

For webcam:

`python3 yolov5_tflite_webcam_inference.py -w yolov5s-fp16.tflite  `

For image:

`python3 yolov5_tflite_image_inference.py -w yolov5s-fp16.tflite -i images/bus.jpg`

For video:

`python3 yolov5_tflite_video_inference.py -w yolov5s-fp16.tflite -v <your video path>`

For folder of images:

`python3 yolov5_tflite_folder_of_images_inference.py -w yolov5s-fp16.tflite -f images/`
