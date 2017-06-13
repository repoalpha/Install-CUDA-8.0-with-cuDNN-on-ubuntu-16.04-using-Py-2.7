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

### Configure jupyter and prompt for password and remember it!

$ jupyter notebook --generate-config

***make a password***

$ jupass=python -c "from notebook.auth import passwd; print(passwd())"
$ echo "c.NotebookApp.password = u'"$jupass"'" >> $HOME/.jupyter/jupyter_notebook_config.py
$ echo "c.NotebookApp.ip = '*'
$ c.NotebookApp.open_browser = False" >> $HOME/.jupyter/jupyter_notebook_config.py

***we will start Jupyter up as follows on the local and remote machines

***Do this from the Local ( the machine with the GPU running the labs)***

$ jupyter notebook --no-browser --port=8889

***Do this from the remote machine you will need to modify the name and ip address***

$ ssh -N -f -L localhost:8888:localhost:8889 machine name@ipaddress

***if you have used a remote machine before for ssh you will need to get into the localhost file and erase the SHA keys against previously used local machines IP addresses.***

### Next we will need to make some changes to the system to get this to work (this is the departure from the conventional installs you may have seen.

***Close off the session jupyter session on the remote, locate the cmd line window running jupyter and ctrl-c.***

***close-off jupiter on the local machine, ctrl-c.***

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

***next restart the local machine jupyter session and remote ssh sessions. I suggest you set up an additional ssh session to the local machine as well as jupyter.***

***on the remote do..***

***ssh your-machine-name@your-ipaddress***

***on the local do...***

$ jupyter notebook --no-browser --port=8889

***Do this from the remote machine, please insert your details your-machine-name@your-ipaddress***

$ ssh -N -f -L localhost:8888:localhost:8889 machine-name@ipaddress

***bring up a http bowser window and put in local machine ipaddress:8889***

$ python
> import keras

/home/sl/anaconda2/lib/python2.7/site-packages/theano/gpuarray/dnn.py:135: UserWarning: Your cuDNN version is more recent than Theano. If you encounter problems, try updating Theano or downgrading cuDNN to version 5.1.
warnings.warn("Your cuDNN version is more recent than "
Using cuDNN version 6020 on context None
Mapped name None to device cuda: GeForce GTX 1080 Ti (0000:02:00.0)
Using Theano backend.

***Just ignore above warning message and re-run line***

$ nvidia-smi

**Next refinements: this is up to you.**

**Create some aliases so you don't have to re-run long instructions on the local and remote.**

***on local machine running our lab and GPU***

$ echo 'alias ju=‘jupyter notebook —-no-browser —-port=8889’' >> ~/.bashrc

$ source ~/.bashrc

***on remote the alias assuming iMAC or similar you will need to modify the .bash_profile or use nano to get into .bash_profile***

***again this is for apple iMAC as client remote session looking at the GPU server!***

$ echo 'alias remote='ssh -N -f -L localhost:8888:localhost:8889 sl@.localdomain' >> ~/.bash_profile

$ source ~/.bash_profile

**Higher risk activities ahead! -you don't need to do this - lets upgrade to nividia driver 378.13**

***close jupyter notebook on the local as this may cause a crash to lightdm , causing guest login re-cycling if you do not gracefully exit jupyter notebook when rebooting.***

***download the nividia driver 378.13 driver from nividia.com and place it in ~/Downloads***

***make sure you are on the remote machine and ssh'd in - this time I really mean you will need to ssh in as you will lose the GUI completely on local. You maybe able to get in on the local cmd line using ctrl+alt+f1 but it does not work always, so ssh.***

$ sudo service lightdm stop

***local screen goes black***

$ sudo init 3

***remove the nvidia 375 driver***

$ sudo apt-get remove --purge nvidia-375 nividia-modprobe nvidia-settings

***that deletes the nvidia 375 driver, now install 378.13***

$ sudo sh NVIDIA-Linux-x86_64-378.13.run

follow screens ignore errors when it asks that the pre-install didn't go well, don't accept 32bit if offers and say yes to X configure and the reboot local.

back in local host check with nvidia-smi restart jupiter on both machines and see how it goes, it may shave a few seconds off.

