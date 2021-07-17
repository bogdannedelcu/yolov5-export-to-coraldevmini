# YoloV5 export to Google Coral Dev Mini board

It is a 2 step process:

1. Flash and update your Coral Dev Mini to the most up to date version
2. Transform the YoloV5s model to Edge Quantized version
3. Run inference on Coral

## ATTENTION

Coral board behaves strange when unplugged without shutdown, corrupting boot filesystem. Since you will be doing a lot of operations I recommend a [USB to Serial](https://coral.ai/docs/dev-board-mini/serial-console/) and monitoring Coral output all the time, this way you will connect easily to the board after flash/restarts.

## Prerequisites for CORAL

To make sure you have the right environment please flash the latest version [2020](https://coral.ai/software/#mendel-dev-board-mini) of the Google Coral Dev Mini board by following the steps [here](https://coral.ai/docs/dev-board-mini/reflash/).

ATTENTION! This will erase completely the board and loose any work you might have. Also this will erase network configuration so you need to follow the [initiation procedure](https://coral.ai/docs/dev-board-mini/get-started/) in order to connect to Wifi.

Once the board is ready with the new version, upgrade all the package on the board as described [here](https://coral.ai/docs/dev-board-mini/get-started/#6-update-the-mendel-software), and beware for system date to be correct.

```
sudo apt-get update

sudo apt-get dist-upgrade

sudo reboot now
```

install requirements.txt

install OpenCV (opencv-python package) for Coral as described in this link: https://krakensystems.co/blog/2020/doing-machine-vision-on-google-coral  I did the steps in the Quick Solution chapter.

ALTERNATIVE: for OpenCV is to take 12 hours to compile it yourself as described in this issue: https://github.com/google-coral/examples-camera/issues/76

## Transform the YoloV5 model to run on EdgeTPU

Use the foollowing [Google Collab](https://colab.research.google.com/drive/1BDX8bjOGyxl6xWFh8mXtheAi36DTa98S?usp=sharing) to transform the PyTorch model to a TFLite quantized. Credits to karanjakhar for the repository used in the Colab.

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
