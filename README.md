# TensorFlow_PanTilt_Cam
Raspberry Pi 3/4 face / objects tracking with Python, OpenCV and Tensorflow, Tensorflow Lite and Coral Edge TPU

FastForward installation guide.

You can choose which framework to use depending either on your hardware (if you have the Coral Edge TPU) or on the
model that you want to use. You can easily use your own trained models.

Features:
- Compatible with Tensorflow, Tensorflow Lite and Edge TPU models
- Select one class from the models to track
- Support all kinds of servos and webcams (not tested on Raspi Cam)

The google's [model zoo](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/detection_model_zoo.md) provides a lot of pretrained models that are sufficient
for many usecases. For example, if you want to track dogs, cats, teddy bears
or humans, the models from the object detection api will work well for you.

# Parts
- Raspberry Pi 3 or 4 (Works on both)
- PCA9685 (~ 6 €)
  - One could connect the servos directly to the raspberry, but there are some downsides. The GPIO of the raspberry uses software PWM instead of hardware PWM which will result in jittering.
  - https://www.amazon.de/dp/B072N8G7Y9/ref=cm_sw_r_cp_api_i_p6WpDb21SD41A

- PAN/Tilt Hat with Servos (7,32€)
  - You could get better, stronger servos, but for a object tracker only moving a webcam these servos are sufficient.
  - https://www.robotshop.com/de/de/pan-tilt-bracket-kit-einzelaufsatz.html

- Step Down DC-DC (6,69€)
  - The advantage of the DC-DC converter is, that you can power the raspberry and the servos from one device. The raspberry 4 should be run with at least 2.5 A input, so this converter will provide sufficient power for both Raspberry and the servos (the RP4 actually can be run on less input power). 
  - 9 ~ 24V Input: Output 5.2V / 6A / 30W 
  - https://www.amazon.de/dp/B071ZRXKJY/ref=cm_sw_r_cp_api_i_rqXpDbZDCPX2N

- Webcam, breadboard, cables etc.


# Installation and configuration
## OS - Raspbian
I decided to use Raspbian Buster as the operating system, as it is easy to install and GPIO will work out of the box.

Simply [download](https://www.raspberrypi.org/downloads/raspbian/) an image of Raspbian and follow the [installation instruction](https://www.raspberrypi.org/documentation/installation/installing-images/README.md). On the date of starting to set up the face-tracking robot (27.07.2019)

Raspbian Buster with desktop and recommended software
Image with desktop and recommended software based on Debian Buster
Version:July 2019
Release date:2019-07-10
Kernel version:4.19
Size: 1945 MB

was the latest version.

After cloning the image to an SD-Card (I highly suggest a fast card like SanDisk Extreme) let Raspberian do all its updates.

 

Next, go to

Preferences --> Raspberry Pi Configuration --> Interfaces and enabel: SSH, VNC and I2C.



Restart and continue updating the OS using

```
sudo apt-get update
sudo apt-get upgrade
```

## Configuring remote access
Luckily, Raspbian comes preinstalled with VNC client remote access.

Install [VNC client](https://www.realvnc.com/de/connect/download/viewer/raspberrypi/) on your main computer and connect to the Raspi via the IP address. e.g. 192.168.178.55

 

(Optional) As my macbook has a smaller screen resolution than my screen connected during OS installation, "automatically scalling" will give poor readability via remote access. We can fix that by setting the resolution of the raspberry match the resolution of your vnc server to use "Scale to 100%" in VNC client.

First we install gedit, a powerfull text editor.

```
sudo apt-get install gedit
```

and open the file with:

```
sudo gedit/boot/config.txt
```

and outcomment and edit the following lines according to your remote master PC/Laptop screen resolution:

> framebuffer_width = 1280

> framebuffer_height = 800

Restart.

## Enable SSH (to use Github/Gitlab)
Creating a SSH key / pair is well described at [gitlab](https://docs.gitlab.com/ee/ssh/).

We use the following command to create a privat and public key on the raspberry:

```
ssh-keygen -t ed25519 -C "email@email.com"
```

On the raspberry the public/private ed25519 key pair will be saved here: **/home/pi/.ssh/id_ed25519**

 

Copy your public SSH key to clipboard:

```
cat < ~/.ssh/id_ed25519.pub (and copy it to the clipboard)
```

 

**Add it to SSH Keys:**

Add your public SSH key to your GitLab account by:

- Clicking your avatar in the upper right corner and selecting Settings.

Navigating to SSH Keys and pasting your public key in the Key field. If you:

- Created the key with a comment, this will appear in the Title field.
- Created the key without a comment, give your key an identifiable title like Work Laptop or Home Workstation.
- Click the Add key button.
 

**Check your ssh connection by**

```
ssh pi@192.168.178.xxx and login into your raspberry
```


## TensorFlow
We will use a prebuilt library of [TensorFlow Lite](https://github.com/PINTO0309/Tensorflow-bin?source=post_page---------------------------) following the tutorial of [Alasdair Allan](https://blog.hackster.io/benchmarking-tensorflow-and-tensorflow-lite-on-the-raspberry-pi-43f51b796796).

Install dependencies
```
sudo apt-get update
sudo apt-get install build-essential
sudo apt-get install git
sudo apt-get install libatlas-base-dev
sudo apt-get install python3-pip
```

I choose TensorFlow 1.14 for python 3.7 as buster includes python 3.7.
```
git clone https://github.com/PINTO0309/Tensorflow-bin.git
cd Tensorflow-bin
pip3 install --user tensorflow-1.14.0-cp37-cp37m-linux_armv7l.whl
```

Check installation:
```
python3 -c "import tensorflow as tf; tf.enable_eager_execution(); print(tf.reduce_sum(tf.random_normal([1000, 1000])))"
```

If everything worked out well you have successfully installed TensorFLow on your Raspberry and should see something like:
> tf.Tensor(-455.19247, shape=(), dtype=float32)

## OpenCV

We will compile opencv 3.4.4 from source mainly following the articles of [Vishwesh Shrimali](https://www.learnopencv.com/install-opencv-3-4-4-on-raspberry-pi/) and [Adrian Rosebrock](https://www.pyimagesearch.com/2018/09/26/install-opencv-4-on-your-raspberry-pi/).

First we remove unnecessary packages for more free space.
```
sudo apt-get purge wolfram-engine
sudo apt-get purge libreoffice*
sudo apt-get clean
sudo apt-get autoremove
```

Then we install some dependencies:
```
sudo apt-get update && sudo apt-get upgrade

sudo apt-get -y install libwebp6 libwebp-dev
sudo apt-get -y install libtiff5 libtiff5-dev
sudo apt-get -y install libjasper1 libjasper-dev
sudo apt-get -y install libilmbase-dev
sudo apt-get -y install libopenexr23 libopenexr-dev
sudo apt-get -y install libgstreamer1.0-0 libgstreamer1.0-dev
sudo apt-get -y install libavcodec-dev
sudo apt-get -y install libavformat-dev 
sudo apt-get -y install libswscale-dev
sudo apt-get -y install libqtgui4
sudo apt-get -y install libqt4-test
sudo apt-get -y install libxine2-dev libv4l-dev

cd /usr/include/linux
sudo ln -s -f ../libv4l1-videodev.h videodev.h

sudo apt-get -y install libgstreamer0.10-dev libgstreamer-plugins-base0.10-dev
sudo apt-get -y install libmp3lame-dev libtheora-dev
sudo apt-get -y install libvorbis-dev libxvidcore-dev libx264-dev
sudo apt-get -y install libopencore-amrnb-dev libopencore-amrwb-dev
sudo apt-get -y install libavresample-dev
sudo apt-get -y install x264 v4l-utils

sudo apt-get -y install libgtk2.0-dev libtbb-dev qt5-default

sudo apt-get -y install libprotobuf-dev protobuf-compiler
sudo apt-get -y install libgoogle-glog-dev libgflags-dev
sudo apt-get -y install libgphoto2-dev libeigen3-dev libhdf5-dev doxygen

sudo apt-get -y install build-essential cmakeunzip pkg-config

sudo apt-get -y install libjpeg-dev libpng-dev libtiff-dev
sudo apt-get -y install libavcodec-dev libswscale-dev libv4l-dev
sudo apt-get -y install libxvidcore-dev libx264-dev
sudo apt-get -y install libgtk-3-dev
sudo apt-get -y install libcanberra-gtk*
sudo apt-get -y install libatlas-base-dev gfortran
sudo apt-get -y install python3-dev
```

Download source and compile:

We will make an installation folder and clone the git repositories:
```
mkdir ~/Installation && cd ~/Installation

git clone https://github.com/opencv/opencv.git
cd opencv
git checkout 3.4
cd ..
 
git clone https://github.com/opencv/opencv_contrib.git
cd opencv_contrib
git checkout 3.4
cd ..
```

The compilation will almost always fail because of a to small swapfile. We will temporarily change the SwapSize from normally 100 MB to 2048 MB until the end of the installation. 

sudo gedit /etc/dphys-swapfile

Change CONF_SWAPSIZE=100 to CONF_SWAPSIZE=2048. 

> CONF_SWAPSIZE=100 >> CONF_SWAPSIZE=1024

Now we need to deactivate the old swap and activate new swap space:
```
sudo /etc/init.d/dphys-swapfile stop
sudo /etc/init.d/dphys-swapfile start
```

![#f03c15](https://placehold.it/15/f03c15/000000?text=+) `Note: Don´t forget to change swap space back to 100 MB after the installation.`


Install numpy and dlib:
```
pip3 install numpy
sudo apt-get install -y libboost-all-dev
pip3 install dlib
```

Navigate to the build directory:
```
cd ~/Installation/opencv
mkdir build
cd build
```

We will make a system wide installation of opencv:

You need to adjust the **path`s**, but if you followed my complete guide everything should be fine.
```
cmake-D CMAKE_BUILD_TYPE=RELEASE \
        -D CMAKE_INSTALL_PREFIX=/usr/local \
        -D INSTALL_C_EXAMPLES=OFF \
        -D OPENCV_ENABLE_NONFREE=ON \
        -D INSTALL_PYTHON_EXAMPLES=OFF \
        -D WITH_TBB=ON \
        -D WITH_V4L=ON \
        -D ENABLE_VFPV3=ON \
        -D ENABLE_NEON=ON \
        -D OPENCV_PYTHON3_INSTALL_PATH=/usr/local/lib/python3.7/site-packages \
        -D WITH_QT=ON \
        -D WITH_OPENGL=ON \
        -D OPENCV_EXTRA_MODULES_PATH=/home/pi/Installation/opencv_contrib/modules \
        -D BUILD_TESTS=OFF \
        -D ENABLE_PRECOMPILED_HEADERS=OFF \
        -D BUILD_EXAMPLES=OFF ..
```

We only use 2 cores for compilation! This takes longer but using all cores sometimes result in failure.
```
make -j2
make install
```

When everything worked out try to import cv2:
```
python3 --> import cv2 --> print("cv2 version:",cv2.__version__)
```

If you see the output 

> cv2 version: 3.4.2

you successfully installed opencv 3.4.2 from source!

If the package is not found, we need to set/add the pythonpath of the installation to ~/.bashrc:
```
sudo gedit ~/.bashrc
export PYTHONPATH=/usr/local/lib/python3.7/site-packages/cv2/python-3.7/cv2/python-3.7
```



