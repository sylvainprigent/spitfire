# SPITFIRe

## Contents

- [Overview](#overview)
- [Repo Contents](#repo-contents)
- [System Requirements](#system-requirements)
- [Installation Guide](#installation-guide)
- [Demo](#demo)
- [Runtime](#runtime)
- [License](./LICENSE)
- [Issues](https://github.com/sylvainprigent/spitfire/issues)
- [Citation](#citation)

# Overview

Modern fluorescent microscopy imaging is still limited by the optical aberrations and the photon budget available
in the specimen. A direct consequence is the necessity to develop flexible and ”off-road” algorithms in order
to recover structural details and improve spatial resolution, which is critical when restraining the illumination to low
levels in order to limit photo-damages. We present a flexible method designed to restore 2D-3D images and videos,
and subtract undesirable out-of-focus background. We assume that the images are sparse and piece-wise smooth,
and are corrupted by mixed Poisson-Gaussian noise. To recover the unknown image, we consider a novel sparsepromoting
regularizer defined as a mixed norm which gathers image intensity and spatial second-order derivatives.
The resulting restoration algorithm named SPITFIR(e) utilizes the primal-dual optimization principle for fast energy
minimization. Experimental results in various microscopy modalities from wide field up to lattice light sheet
demonstrate the ability of the SPITFIR(e) algorithm to efficiently reduce noise, blur, and out-of-focus background,
while avoiding the emergence of deconvolution artifacts.

# Repo Contents

We provide 2 implementations of `SPITFIRe`. One is the original c++ implementation and works for 2D, 3D and 4D images. The second is a python pyTorch based implementation. 

## SPITFIRe C++

A c++ implementation of `SPITFIRe` is available in the [SimgLib](https://github.com/sylvainprigent/simglib) repository as a stand alone c++ command line interface. It is implemented for CPU and for GPU (cuda). 

## SPITFIRe python

A `SPITFIRe` python implementation using the pyTorch library is meant to provide an easy to use and modify full python implementation of the `SPITFIRe` regularizer. It is available in the [sdeconv](https://github.com/sylvainprigent/sdeconv) package.

# System Requirements

## Hardware Requirements

The `SPITFIRe` algorithm requires only a standard computer with enough RAM proportional to the input data size. Most of the experiment we made where done on a computer with about 8 GB of RAM. For optimal performance, we recommend a computer with the following specs:

RAM: 16+ GB  
CPU: 4+ cores, 3.3+ GHz/core

The cuda `SPITFIRe` version needs a NVIDIA graphic card and the cuda drivers.

## Software Requirements

### OS Requirements

The `SPITFIRe` software is compatible with *Windows 10*, *MacOS* and *Linux* operating systems. The developmental version of the package has been tested on the following systems:

- Linux: Ubuntu 21.04 
- Mac OSX: Mac OS Catalina 10.15.11    
- Windows: 10 

# Installation Guide

Depending on the needed version they are several methods to install `SPITFIRe`. We recomand using the napari plugin or the `sdeconv` python library since it is available in PyPI 

## Using napari

The easiest method to install the napari plugin is using Conda.

```shell
conda create -y -n napari-env -c conda-forge python=3.9 pip
conda activate napari-env
python -m pip install "napari[all]"
python -m pip install napari-sdeconv
```

Napari can then be started with:

```shell
conda activate napari-env
napari
```
and `SPITFIRe` is in the plugin menu

## Using the python version

The python primal-dual version can be installed using Conda

```shell
conda create -y -n spitfire -c conda-forge python=3.9 pip
conda activate spitfire
python -m pip install sdeconv
```

The `SPITFIRe` python class can then be used as follow:

```python
from sdeconv.data import celegans
from sdeconv.deconv import Spitfire, SPSFGaussian

# load the blurry image
image = celegans()

# create a PSF
psf_generator = SPSFGaussian((1.5, 1.5), (13, 13))
psf = psf_generator()

filter_ = Spitfire(psf, weight=0.6, reg=0.995, gradient_step=0.01, precision=1e-7, pad=13)
deconv_image = filter_(image)
```

## Using the c++ CLI

The C++ implementation of `SPITFIRe` is provided with a Command Line Interface. 

A pre-compiled package is provided in the conda package simglib:
```
conda install -c sylvainprigent simglib
```

To compile `SPITFIRe` from C++ source code, `cmake` is needed. For people using windows, we recommand using Conda to manage cmake and the dependencies (libtiff, fftw3) and the visual compiler. The main step are (in unix command):

```
git clone https://github.com/sylvainprigent/simglib
cd simglib
mkdir build
cd build
cmake .. -DSL_USE_OPENMP=ON -Dsimglib_BUILD_CUDA=ON -Dsimglib_BUILD_TOOLS=ON
make
make install
```

Then the `SPITFIRe` CLI tool can be called as follows for the CPU version:
```
simgspitfiredeconv2d -i blurry_image.tif -o deblurred_image.tif -sigme 1.5 -regularization 12 -weighting 0.6 -niter 200
```
and as follow for the GPU version:
```
simgcuspitfire2ddeconv -i blurry_image.tif -o deblurred_image.tif -sigma 1.5 -regularization 12 -weighting 0.6 -niter 200
```

# Demo

The demo using the graphical interface can be found in the [napari plugin page](https://www.napari-hub.org/plugins/napari-sdeconv) 

# Run time


| SPITFIR(e) 200 iterations   |      Image size      |  CPU (2.8 GHz, 4 threads) | GPU (Quadro M2000) | GPU (Tesla K80)|
|----------|:-------------:|------:|------:|------:|
| 2D Denoising |  512x512 | 349ms |  345ms | 278ms |
| 3D Denoising |    512x512x22  | 8s 74ms |  3s 78ms | 1s 277ms |
| 4D Denoising | 512x512x14x32 | 8min 23s 789ms | 1min 40s 98ms | 45s 744ms |
| 2D Deconvolution | 512x512 | 767ms | 550ms | 428ms |
| 3D Deconvolution | 512x512x22 | 23s 574ms | 5s 786ms | 2s 713ms |
