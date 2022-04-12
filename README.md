# Building Python 3.10 and OpenCV 4.5.5 on Jetson Nano

To install Python 3.10 and OpenCV 4.5.5 on Jetson Nano, it's better to use 
a new installation of JetPack 4.6 or higher. This tutorial it's about to build 
Python 3.10 and OpenCV to use the latest python version and OpenCV with CUDA 
support in our projects.

--------------------------------------------------------------------------------------------------------------------
NOTE: This tutorial is based on this original tutorial from Qengineering, this varian is only for the case of 
work with Python3.10 and OpenCV 4.5.5. If you are ok with install OpenCV under Python3.6, please follow the 
original tutorial.

https://qengineering.eu/install-opencv-4.5-on-jetson-nano.html

--------------------------------------------------------------------------------------------------------------------

Before we start, we need to install the required packages.

```bash
sudo apt-get install build-essential cmake git unzip pkg-config
```

## Before start the building process, we need to increase the memory swap limit

Nano text editor is a good choice to edit this file.
```bash
sudo apt-get install nano
```

To increase the swap we need to edit the `/etc/dphys-swapfile` file.

```bash
sudo apt-get install dphys-swapfile

# Pending to edit the file
sudo nano /sbin/dphys-swapfile
sudo nano /etc/dphys-swapfile

# reboot
sudo reboot
```

## Building Python 3.10

NOTE: Please don't remove the previous installation of Python 3.6 included in the
JetPack, other native packages from Nvidia are only available in this version.

To build Python 3.10, you need to install the following dependencies:

```bash
sudo apt install zlib1g-dev \
  libncurses5-dev \
  libgdbm-dev \
  libnss3-dev \
  libssl-dev \
  libreadline-dev \
  libffi-dev \
  libsqlite3-dev \
  libbz2-dev

```

To download the source code of Python 3.10 and build it, you need to run the next command:

```bash
wget https://www.python.org/ftp/python/3.10.4/Python-3.10.4.tar.xz
tar xf Python-3.10.4.tar.xz
cd Python-3.10.4
./configure --enable-optimizations
```

Run make with all active cores for Jetson Nano:
```bash
make -j$(nproc)
```

Install into the alternate location for this version of Python like `/usr/local/bin/python3.10`.

```bash
sudo -H make altinstall
```

Now you have Python 3.10 installed. You can run the following command to check if it's working:

```bash
python3.10 -V
```

You will need to install `numpy` in python 3.10 globally.

```bash
sudo python3.10 -m pip install numpy
```

The site-package directory is `/usr/local/lib/python3.10/site-packages`, here is where the packages are installed, you'll 
need this path in the OpenCV building process.

### Change the default Python version

To change the default Python version, you need to run the following command:

```bash
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.6 1
sudo update-alternatives --install /usr/local/bin/python3 python3 /usr/local/bin/python3.10 2
sudo update-alternatives --set python3 /usr/local/bin/python3.10
```

This way, you can maintain the previous python version (3.6) and the new one (3.10) as default.

## Building OpenCV 4.5.5

To build OpenCV 4.5.5, you need to install the following dependencies:

```bash
sudo apt-get install \
  libjpeg-dev \
  libpng-dev \
  libtiff-dev \
  libavcodec-dev \
  libavformat-dev \
  libswscale-dev \
  libgtk2.0-dev \
  libcanberra-gtk* \
  libxvidcore-dev \
  libx264-dev \
  libgtk-3-dev \
  libtbb2 \
  libtbb-dev \
  libdc1394-22-dev \
  libv4l-dev \
  v4l-utils \
  libavresample-dev \
  libvorbis-dev \
  libxine2-dev \
  libfaac-dev \
  libmp3lame-dev \
  libtheora-dev \
  libopencore-amrnb-dev \
  libopencore-amrwb-dev \
  libopenblas-dev \
  libatlas-base-dev \
  libblas-dev \
  liblapack-dev \
  gfortran \
  libhdf5-dev \
  protobuf-compiler \
  libprotobuf-dev \
  libgoogle-glog-dev \
  libgflags-dev -y
```

Download the source code of OpenCV 4.5.5 and OpenCV-contrib 4.5.5:

```bash
cd ~ 
wget -O opencv.zip https://github.com/opencv/opencv/archive/4.5.5.zip 
wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.5.5.zip 
```

Now, unpacking the source code:

```bash
# unzip files
unzip opencv.zip 
unzip opencv_contrib.zip

# change the name of the folders
mv opencv-4.5.5 opencv
mv opencv_contrib-4.5.5 opencv_contrib

# remove the zip files
rm opencv.zip
rm opencv_contrib.zip
```

Make a build directory:

```bash
cd ~/opencv
mkdir build
cd build
```

CMake is a tool that helps you to configure and build software. It is a cross-platform, open-source, 
open for extension, and open for modification (CMake) build system.

```bash
cmake \
    -D CMAKE_BUILD_TYPE=RELEASE \
    -D CMAKE_INSTALL_PREFIX=/usr \
    -D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib/modules \
    -D EIGEN_INCLUDE_PATH=/usr/include/eigen3 \
    -D WITH_OPENCL=OFF \
    -D WITH_CUDA=ON \
    -D CUDA_ARCH_BIN=5.3 \
    -D CUDA_ARCH_PTX="" \
    -D WITH_CUDNN=ON \
    -D WITH_CUBLAS=ON \
    -D ENABLE_FAST_MATH=ON \
    -D CUDA_FAST_MATH=ON \
    -D OPENCV_DNN_CUDA=ON \
    -D ENABLE_NEON=ON \
    -D WITH_QT=OFF \
    -D WITH_OPENMP=ON \
    -D BUILD_TIFF=ON \
    -D WITH_FFMPEG=ON \
    -D WITH_GSTREAMER=ON \
    -D WITH_TBB=ON \
    -D BUILD_TBB=ON \
    -D BUILD_TESTS=OFF \
    -D WITH_EIGEN=ON \
    -D WITH_V4L=ON \
    -D WITH_LIBV4L=ON \
    -D OPENCV_ENABLE_NONFREE=ON \
    -D INSTALL_C_EXAMPLES=OFF \
    -D INSTALL_PYTHON_EXAMPLES=OFF \
    -D PYTHON3_PACKAGES_PATH=/usr/local/lib/python3.10/site-packages \
    -D OPENCV_GENERATE_PKGCONFIG=ON \
    -D BUILD_EXAMPLES=OFF \
    -D PYTHON_DEFAULT_EXECUTABLE=$(which python3) ..
```

After configuring, you can run the build process:

```bash
make -j$(nproc)
```

After compiling, you can install OpenCV:

```bash
sudo rm -r /usr/include/opencv4/opencv2
sudo make install
sudo ldconfig

# cleaning (frees 300 MB)
make clean
sudo apt-get update
```

And now it's not more required the swap increment, you can back to the original swap size:

```bash
sudo /etc/init.d/dphys-swapfile stop
sudo apt-get remove --purge dphys-swapfile
```

And remove the downloaded files:

```bash
sudo rm -rf ~/opencv
sudo rm -rf ~/opencv_contrib
```

After install opencv in the system, you'll need to copy a pkgconfig file generated by the build process to the 
system's `/usr/lib/pkgconfig` directory.

```bash
sudo cp ./unix-install/opencv4.pc /usr/lib/pkgconfig/opencv4.pc
```

This way yoy can compile with opencv easily using:

```bash
`pkg-config --cflags --libs opencv4`
```

...

## Testing installation

To test if it's working just try:

```bash
echo `pkg-config --cflags --libs opencv4`
```

And you should see all the OpenCV include directories and the libraries.

To test the python installation, you can run the following command:

```bash
python3 -c "import cv2; print(cv2.__version__)"
```

And it should print the version of OpenCV.

```
4.5.5
```
