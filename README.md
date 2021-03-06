# YoloV5 export to Google Coral Dev Mini board

It is a 2 step process:

1. Flash and update your Coral Dev Mini to the most up to date version
2. Transform the YoloV5s model to Edge Quantized version
3. Run inference on Coral

## ATTENTION

Coral board behaves strange when unplugged without shutdown, corrupting boot filesystem. Since you will be doing a lot of operations I recommend a [USB to Serial](https://coral.ai/docs/dev-board-mini/serial-console/) and monitoring Coral output all the time, this way you will connect easily to the board after flash/restarts.

### Add SWAP!
Coral board has only 2G of RAM and 8G of storage. I strongly recommend to download the packages on a USB stick and add a swap file to the board on a separate SD card. 

To add swap file to Coral Dev Mini please follow the instructions bellow.
https://github.com/f0cal/google-coral/issues/61

### Add SPACE
I used a 8GB USB stick and another 8GB SD card for Swap. If you get disk full messages I recommend to restart that operation from scratch after adding more space to the device.


## Prerequisites for CORAL

To make sure you have the right environment please flash the latest version [2020](https://coral.ai/software/#mendel-dev-board-mini) of the Google Coral Dev Mini board by following the steps [here](https://coral.ai/docs/dev-board-mini/reflash/).

ATTENTION! This will erase completely the board and loose any work you might have. Also this will erase network configuration so you need to follow the [initiation procedure](https://coral.ai/docs/dev-board-mini/get-started/) in order to connect to Wifi.

Once the board is ready with the new version, upgrade all the package on the board as described [here](https://coral.ai/docs/dev-board-mini/get-started/#6-update-the-mendel-software), and beware for system date to be correct.

```
sudo apt-get update

sudo apt-get dist-upgrade

sudo reboot now
```

### OpenCV package

The library opencv-python does not install by default on Coral so you must follow the steps bellow in order to succeed.

install OpenCV (opencv-python package) for Coral as described in this link: https://krakensystems.co/blog/2020/doing-machine-vision-on-google-coral  I did the steps in the Quick Solution chapter.

ALTERNATIVE: for OpenCV is to take 12 hours to compile it yourself as described in this issue: https://github.com/google-coral/examples-camera/issues/76

### PyTorch package

To build pytorch as tutorial (24h in my case) my advice is to  use "screen" to keep a long running session open.

so: 
```
sudo apt-get install screen
```
Once you start a screen use Ctrl-A + D to disconnect, then
```
screen -r 
```

to reconnect a running session.

Now prepare for a 24 hours journey, start your screen and a separate shell to monitor the board.

Setting MAX_JOBS  will enable only 3 parallel CCL to be executed in the installation of python packages. In my tests this is confortable for the 2G memory available.

Credits to 
https://stackoverflow.com/questions/63047424/how-to-install-pytorch-in-coral-dev-board, I just added the EXPORT command.
```
export MAX_JOBS=3
# clone pyTorch repo
git clone http://github.com/pytorch/pytorch
cd pytorch
git submodule update --init --recursive

# install prereqs
sudo pip3 install -U setuptools
sudo apt-get install python3-pycoral
```

Now start building TORCH with
```
python3 setup.py build
```

ATTENTION: Monitor the build in a separate shell with TOP to see if your progress stops. In my case around 62% a file named RegisterMeta.cpp will freeze as not enough RAM is abailable. Ctrl+C to interrupt and set this env variable to 1.
```
export MAX_JOBS=1
```

Restart the process, will continue where it left
```
python3 setup.py build
python3 setup.py install
```

After some time it compiles and reaches batch_norm_kernel.cpp file so please set back MAX_JOBS = 3 to speedup process, or leave it like this as it will take very long time.


Install other requirements

```
pip3 install numpy
pip3 install pandas # prepare to wait 3 hours
pip3 install tqdm # aprox 10 minutes
```

Install Torch Vision

```
sudo pip3 install matplotlib # aproxx 20 min
sudo pip3 install seaborn
sudo pip3 install scipy==1.4.1

```

```
sudo pip3 install ninja # this should take 3-45 minutes
```

Again, please set MAX_JOBS=2 because Ninja will propose a too large value in my opinion for the available RAM and monitor with TOP

```
Git clone https://github.com/pytorch/vision
cd vision
python3 setup.py install # this should take about 1 hour
```

# Run the inference on Google Coral Mini

## Download pre-trained Quantized EdgeTPU weights cooked by me
After tests it seems only the 96x96 image size fits the EdgeTPU memory on all operations. This however is giving low quality in my tests.
```
git clone https://github.com/bogdannedelcu/yolov5-export-to-coraldevmini
git clone https://github.com/bogdannedelcu/yolov5/
cd yolov5
git checkout tf-edgetpu


```
Now copy the model and the images in the yolov5 folder and start detecting:
```
detect.py --weight yolov5s-int8_edgetpu_416.tflite --img 416 --tfl-int8 --tfl-detect --source bus.jpg --edgetpu
```

DISCLAIMER: I updated the tf-edgetpu branch of the YoloV5 repository to use only tflite and pycoral packages, as tensorflow is not available on the board.

## Transform the YoloV5 model to run on EdgeTPU (OPTIONAL)

Use the foollowing [Google Collab](https://colab.research.google.com/drive/1BDX8bjOGyxl6xWFh8mXtheAi36DTa98S?usp=sharing) to transform the PyTorch model to a TFLite quantized.

# My results on the Codal Dev Mini Board in seconds

| Image file  | Time on CPU F16 | Time on EdgeTPU F16  | Time on EdgeTPU Quantized 416px | Time on EdgeTPU Quantized 224px |
|---|---|---|---| --- |
| bus.jpg - run once|  3.2 | 6.2  | 0.6  |  0.3 |
| bus.jpg - run 100 times|  3.2 | 6.2  | 0.38  |  0.14 |

* time averaged after 1 and respective 100 interpreter.invoke() calls

My conclusion, on a Coral Dev Board Mini EdgeTPU Quantized model compiled and run we get :
- at 224x224 image size a 7FPS
- at 416x416 image size a 2.6FPS (less operations run on EdgeTPU than at 224x224)

I included in the repository some images of the output on 224 and 416 pixels and also the conversion report from edgetpu_compiller for both versions. There are some layers in the NN which are not converted due to compatibility and size (416 is worse than 224)
