# This instruction is designed to work with ubuntu 16.04 LTS
**I hope this instruction can be of help to install CUDA 8.0 using cuDNN 6020 on ubuntu using python 2.7, which is not an easy exercise as compared to using cuDNN 5.1.**
**The backend assumes theano 0.9 and the front end assumes keras**

### INSTALL SSH OPENSERVER

$ sudo apt-get install openssh-server
$ sudo service ssh status

ctrl-c to get out

***locate your ip address and write it down to use on your other machine.***

$ ifconfig

***log into your local machine running the lab and GPU using ssh your-machine-name@your-ipaddsress***

***ensure system is updated and has basic build tools***

$ sudo apt-get update
$ sudo apt-get --assume-yes upgrade
$ sudo apt-get --assume-yes install tmux build-essential gcc g++ make binutils
$ sudo apt-get --assume-yes install software-properties-common

### Download and install GPU drivers CUDA 8.0.61

$ wget "http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/cuda-repo-ubuntu1604_8.0.61-1_amd64.deb" -O "cuda-repo-ubuntu1604_8.0.61-1_amd64.deb"

$ sudo dpkg -i cuda-repo-ubuntu1604_8.0.61-1_amd64.deb
$ sudo apt-get update
$ sudo apt-get -y install cuda
$ sudo modprobe nvidia
$ nvidia-smi

### Download cuDNN 6.0 from nvidia developers website https://developer.nvidia.com/1, you will need an account, sign up. Obtain the files for Linux cuDNN 6.0 for CUDA 8.0 and place that in your ~/Downloads folder.

### INSTALL cuDNN 6.0

$ tar -zxf cudnn-8.0-linux-x64-v6.0.tgz
$ cd cuda
$ sudo cp lib64/* /usr/local/cuda/lib64/
$ sudo cp include/* /usr/local/cuda/include/

### Install Anaconda for current user

$ mkdir downloads
$ cd downloads
$ wget "https://repo.continuum.io/archive/Anaconda2-4.2.0-Linux-x86_64.sh" -O "Anaconda2-4.2.0-Linux-x86_64.sh"
$ bash "Anaconda2-4.2.0-Linux-x86_64.sh" -b

$ echo "export PATH=\"$HOME/anaconda2/bin:\$PATH\"" >> ~/.bashrc
$ export PATH="$HOME/anaconda2/bin:$PATH"
$ conda install -y bcolz
$ conda upgrade -y --all

### Install and configure theano 

$ pip install theano
$ echo "[global]
device = gpu
floatX = float32
[cuda]
root = /usr/local/cuda" > ~/.theanorc

### Download the libgpuarray libraries key to getting this going.

$ git clone https://github.com/Theano/libgpuarray.git1
$ cd libgpuarray

$ mkdir Build
$ cd Build
$ cmake .. -DCMAKE_BUILD_TYPE=Release
$ make
$ make install
$ cd ..

### Change the .theanorc file 
***which should be in the home directory cd ~ but is hidden so use ls -a to find file:***
***Use nano or vi to change***

[global]
device = cuda
floatX = float32

[cuda]
root = /usr/local/cuda

### Finally install pygpu using conda 
***this piece is also essential and done much later at this point of the install***

$ conda install pygpu

***It's probably a good time for a reboot so restart the local***

$ python
$>>> import keras

/home/sl/anaconda2/lib/python2.7/site-packages/theano/gpuarray/dnn.py:135: UserWarning: Your cuDNN version is more recent than Theano. If you encounter problems, try updating Theano or downgrading cuDNN to version 5.1.
warnings.warn("Your cuDNN version is more recent than "
Using cuDNN version 6020 on context None
Mapped name None to device cuda: GeForce GTX 1080 Ti (0000:02:00.0)
Using Theano backend.

***Just ignore above warning message and re-run line***

$ nvidia-smi

### Higher risk activities ahead! -you don't need to do this - this section upgrades to nividia driver 378.13

***download the nividia driver 378.13 driver from nividia.com and place it in ~/Downloads***

***make sure you are on the remote machine and ssh'd in - You will definiateley need to ssh in to the server as you will lose the GUI completely as local user. You maybe able to get in on the local cmd line using ctrl+alt+f1 but it does not work always, so ssh.***

$ sudo service lightdm stop

***local screen goes black***

$ sudo init 3

***remove the nvidia 375 driver***

$ sudo apt-get remove --purge nvidia-375 nividia-modprobe nvidia-settings

***that deletes the nvidia 375 driver, now install 378.13***

$ sudo sh NVIDIA-Linux-x86_64-378.13.run

follow screens ignore errors when it asks that the pre-install didn't go well, don't accept 32bit if offers and say yes to X configure and the reboot local.

back in local host check with nvidia-smi restart jupiter on both machines and see how it goes, it may shave a few seconds off.

