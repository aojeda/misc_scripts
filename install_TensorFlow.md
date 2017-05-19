# How to configure and install TensorFlow
The following tutorial shows how to configure an Ubuntu  14.04 system with NVIDIA graphic card to run TensorFlow (see more details [here](https://www.tensorflow.org/install/install_linux#InstallingNativePip)). All commands should be typed on a Bash terminal.

1- Download cuda
```bash
wget https://developer.nvidia.com/compute/cuda/8.0/Prod2/local_installers/cuda-repo-ubuntu1404-8-0-local-ga2_8.0.61-1_amd64-deb
```

2- Install cuda
```bash
dpkg -i cuda-repo-ubuntu1404-8-0-local-ga2_8.0.61-1_amd64-deb
apt-get update
apt-get install cuda
```

3- Install CUDA Profile Tools Interface
```bash
sudo apt-get install libcupti-dev
```

4- Environment Setup. Use a text editor to add the following lines to `/etc/bash.bashrc `
```bash
export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```