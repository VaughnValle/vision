---
tags: 
- Documentation
- vision
date:  08-11-22
project: vision
---

# VISION: Recreating the Project
###### Visually Impaired-specific System for Indoor and Outdoor Navigation

by Vaughn Matthew Q. Valle / August 11, 2022

---

> [!abstract] Abstract
> Visual perception is one the most important cognitive processes --  the process of absorbing what one sees, organizing it in the brain, and making sense of it. However, not everyone has the luxury of the sense of sight. Roughly two (2) billion people around the world suffer from vision impairment – 39 million people of which are completely blind and eighty-two percent (82%) of which are fifty (50) years and older (World Health Organization, 2015).
> <br> <br>
> VISION is an innovative device that combines the power of local deep learning and image processing to obtain, analyze, and process real-time visual information around the user. It aims to be the “smart companion” for the blind - to process critical visual information around the user’s environment where the blind cannot and puts heavy emphasis on being a device that enables the user to freely roam the world and accomplish day-to-day tasks independently.

### Prerequisites
- **Laptop**
	- for **Windows**, install an SSH client such as [Putty](https://the.earth.li/~sgtatham/putty/latest/w64/putty-64bit-0.77-installer.msi) and [Bonjour](https://downloads.digitaltrends.com/bonjour/windows/post-download#) for local hostname
	- for **Linux** and **Mac**, download your respective SSH client via terminal
- **Raspberry Pi 3B+ SBC** running _Raspi OS Lite_
- **Intel Neural Compute Stick 2**
- **PiCamera v2**
- **8GB or higher microSD card** preferably _class 10_
- **5V Power Supply** (2.5A or higher supply is recommended)

### Setup
#### Flashing Raspi OS with SSH access
Install [Raspberry Pi Imager](https://www.raspberrypi.com/software/) on your computer by clicking the Download button of your respective platform. Run the installer and save the program to a location of your choice.

Once installation has finished, open the program. Select **Raspberry Pi OS Lite (64-bit)** as the Operating System and your microSD Card as Storage. 

>[!warning] WARNING! DO NOT click on **WRITE** yet!
>Doing so will flash the image without SSH being setup and you will not have access to the RPi remotely.

Instead, click on the gear icon located on the lower left of the window. This will open up the imager's settings.

![[Pasted image 20220811171856.png]]

> [!help] If you don't see it, hold **Ctrl + Shift + X** instead

Check *Enable SSH* and *Use password for authentication*, *Configure wireless LAN*, and *Set locale settings*. Then modify the input fields according to your WiFi's name and password as well as  your country, time zone, and keyboard layout. I also recommend you **change your username and password** to prevent unauthorized SSH access in the future. 

Once finished, click on **Save**, then **Write** and wait for it to finish flashing. Once it's done eject the microSD card, and insert it into your RasPi board.

#### Connecting the Hardware

Start by connecting your *Intel NCS2 stick* to your RasPi Board. Connect it to one of the USB ports at the back of the board just as you would with a thumb drive. Then connect your *PiCamera v2* to the Camera port of your RasPi Board.

> [!info]  NOTE: Camera port is located to the left of the audio jack
> Lift the black tab and insert the camera's ribbon cable with the exposed pins **face down** and in contact with the white plastic connector side. Firmly press the black tab back into place afterwards. See the image below for reference.

![[Pasted image 20220811183620.png]]

To prevent short circuits, place your RasPi board on a non-conductive surface especially when it doesn't have a case. Then plug your RasPi board to the power supply.

#### Accessing the RasPi via SSH

Connect your laptop to the same network you entered in the imager's settings.

Wait for about a minute after plugging the board to the power supply. Then follow the next steps depending on what platform your laptop is using.

##### For Windows:

Launch PuTTY on Windows. Enter `raspberrypi.local` as Host Name and make sure **SSH** and port **22** are selected. Then click on *Open*

##### For Linux and Mac:

Open your terminal and type `ssh pi@raspberrypi.local`
<br>
<br>
<br>
If prompted for the RSA key fingerprint, click **Accept** *for Windows* or type `yes` *for Mac and Linux*

Once SSH connection is established, you will then be prompted for the RasPi's username and password. If you didn't change it, the default credentials are as follows:

> [!info] Default login credentials
username: `pi` <br>
password: `raspberry`

Hit Enter and you should now be connected to the RasPi via SSH.

#### Expand Filesystem

Some filesystems in a fresh install don't use all the available space on the microSD card. 

To check filesystem usage, execute `df -h` in your PuTTY window or terminal. If you see that **Avail** under the `/dev/root` filesystem to be too small for your microSD card, it's likely that we need to expand it to fully utilize the microSD card storage. 

To do that, execute `sudo raspi-config`. Then navigate to `Advanced Options => Expand Filesystem => Finish`. 

Once done, reboot the RasPi by executing `sudo reboot`

Connect to your RasPi via SSH by following [[#Accessing the RasPi via SSH]] again.

#### Cloning the Codebase Repository from GitHub

Install Git by executing `sudo apt install git`. Then clone the GitHub repo to a folder (here we're using the Downloads folder)

```bash
cd ~/Downloads && git clone https://github.com/VaughnValle/vision.git
```

#### Installing Dependencies

Before we can replicate the original software environment, we need to install some packages both OpenVINO and OpenCV need to run.

First, update your repo and upgrade your RasPi system via `sudo apt update && sudo apt upgrade`. Then execute the commands below to install all the dependencies we need.

>[!tip] Command to install dependencies
>```bash
>sudo apt install build-essential cmake unzip pkg-config libjpeg-dev libpng-dev libtiff-dev libavcodec-dev libavformat-dev libswscale-dev libv4l-dev libxvidcore-dev libx264-dev libgtk-3-dev libcanberra-gtk* libatlas-base-dev gfortran python3-dev
>```

#### Configuring OpenVINO on Raspberry Pi OS

Unpack the OpenVINO toolkit for the GitHub repo
```bash
cd ~/Downloads/vision
tar -xf l_openvino_toolkit_runtime_raspbian*
mv l_openvino_toolkit_runtime_raspbian* ~/openvino
```

Load OpenVINO on any TTY instance/login

```bash
echo -e "\n# OpenVINO Setupvars\nsource ~/openvino/bin/setupvars.sh" >> ~/.bashrc
source ~/.bashrc
```

Configure USB Dev rules to allow the Intel NCS2 stick to connect via USB

```bash
sudo usermod -a -G users "$(whoami)"
cd ~ && sh openvino/install_dependencies/install_NCS_udev_rules.sh
```

#### Setting up OpenVINO Python Environment

Get Python package manager PIP
```bash
cd ~/Downloads
wget https://bootstrap.pypa.io/get-pip.py
sudo python3 get-pip.py
```

Install *virtualenv* and *virtualenvwrapper* for setting up Python virtual environments
```bash
sudo pip install virtualenv virtualenvwrapper
sudo rm -rf ~/Downloads/get-pip.py ~/.cache/pip
```

Append virtualenv startup commands to bashrc

```bash
echo -e "\n# Virtualenv" >> ~/.bashrc
echo "export WORKON_HOME=$HOME/.virtualenvs" >> ~/.bashrc
echo "export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3" >> ~/.bashrc
echo "source /usr/local/bin/virtualenvwrapper.sh" >> ~/.bashrc
echo "VIRTUALENVWRAPPER_ENV_BIN_DIR=bin" >> ~/.bashrc
source ~/.bashrc
```

Create a virtual environment named *vision* for our project
```bash
mkvirtualenv vision -p python3
```

Install packages in our python OpenVINO virtual environment
```bash
workon vision
pip install numpy "picamera[array]" imutils
```

#### Creating a Bash script for OpenVINO startup

For ease of use, we can automate initializing and working in our python virtual environment by creating a bash script with the following lines
```bash
#!/bin/bash
echo "Starting Python with OpenCV-OpenVINO"
source ~/openvino/bin/setupvars.sh
workon openvino
```

However, I added this file to the GitHub repo already so copy it to your home directory with 
```bash
cp ~/Downloads/vision/startup_vision.sh ~
```


If you want to test if everything is setup correctly:
>[!info] Testing if environment is setup correctly
>Run the script we made with `~/startup_vision.sh`.  Then run python with `python3`

Then run:
```python
import cv2
cv2.__version__
```
You should see one of the following outputs:

> [!check] 'x.x.x-openvino'
> Congratulations! You have successfully setup the environment correctly. <br><br>
> Your RasPi is now ready to run python code with OpenCV and OpenVINO integration.

> [!fail] ModuleNotFoundError: No module named 'cv2'
> If you see this error, this means Python in your environment cannot locate your OpenCV or OpenVINO installation. It is likely that something went wrong or you missed a step in the previous sections. <br><br>
> Double check that you followed all the steps in [[#Setting up OpenVINO Python Environment]]

If you have ensured that all the steps were followed yet the/an error still occurs, feel free to submit a new issue in this [GitHub Repo's Issue page](https://github.com/VaughnValle/Vaughn-obsidian/issues)

### Operation

First, make sure your RasPi is powered up and connected to the same WiFi network. Then login via SSH using the instructions provided in the previous section, [[#Accessing the RasPi via SSH]].

Initialize our Python environment with `~/startup_vision.sh`.

Attach an audio device to your RasPi's audio jack and run the live object detection code with
```bash
python3 openvino_headless.py --prototxt MobileNetSSD_deploy.prototxt --model MobileNetSSD_deploy.caffemodel
```
<br>

#### Frequently Asked Questions

---
#### PuTTY
PuTTY is an SSH client for the Windows platform and allows us to connect to the RasPi board via SSH from the laptop, without the need of external components such as a monitor, keyboard, and mouse
##### Bonjour
Bonjour is a program by Apple Inc. that allows for zero-configuration (*zeroconf*) networking between different types of devices. In this case, it allows us to communicate to the RasPi via `raspberrypi.local` as the hostname, without the need to get the Pi's IP address.
#### GitHub
GitHub is a code hosting platform for collaboration. It uses the program **Git** to help developers store, manager, track, and control changes to their software.

[Visit Vaughn Valle's GitHub page](https://github.com/VaughnValle/)
<br>

[VISION GitHub page](https://github.com/VaughnValle/vision)

#### How to Shutdown?
Run `sudo shutdown -h now`
#### How to Reboot?
Run `sudo reboot`

