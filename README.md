# Event-based Science Camera for Asteroid Particle Ejections (ESCAPE)

This repository contains the development code for the event-based science project on particle tracking near active asteroids detailed in:

> Loïc J. Azzalini and Dario Izzo.
Tracking Particles Ejected From Active Asteroid Bennu with Event-Based Vision. In *Proceedings of the XXVII AIDAA International Congress*, 2023. Work in progress

The repository will be updated shortly after the conference.

## Renders

![PANGU frame](./media/ejecta3x.png)|
:-------------------------:|
Reconstruction of a 14-particle ejection episode: Blender render (a) and the corresponding event-based representation, with (b) and without (c) noise |

## Dependencies

The following dependencies are needed to run the ESCAPE project.

### SPICE kernels

Bennu's dynamics and particle ejection events can be looked up in the SPICE kernels corresponding to the `osiris-rex` mission. The `common` and `osirirs_rex` kernels listed in `_kernels/kernel_meta.txt` can be found in the [main SPICE kernel respository](https://naif.jpl.nasa.gov/pub/naif/). We recommend using the following folder structure:
```
|-- _kernels
|   |-- common
|   |   |-- fk
|   |   |-- lsk
|   |   |-- pck
|   |   |-- spk
|   |-- osiris_rex
|   |   |-- dsk
|   |   |-- spk
```
The kernels specific to the particles, `particles_pub_03Mar2020.bsp` are taken from [Hergenrother et al. (2020)](https://agupubs.onlinelibrary.wiley.com/doi/10.1029/2020JE006381) and can be found in the publication's [supplementary material](https://figshare.com/articles/online_resource/Supplementary_Data_for_Bennu_Particle_Analysis/11328398).

In order to extract the comments from the kernel file `particles_pub_03Mar2020.bsp`, the SPICE utility `commnt` needs to be downloaded from the [main repository](https://naif.jpl.nasa.gov/pub/naif/toolkit/C/) into the directory `_kernels/osiris_rex/spk`. Then, run
```
chmod u+x commnt
./commnt -e particles_pub_03Mar2020.bsp particles_pub_03Mar2020_comments.txt
```

### Scene generation

Blender is used to generated photorealistic renders of the asteroid Bennu during particle ejection episodes, as seen by the OSIRIS-REx spacecraft NavCam 1 cameras. Download and install [Blender 3.3 LTS](https://www.blender.org/download/lts/3-3/) and save the directory with the executable (e.g., `/opt/blender/`, `/Applications/Blender.app/Contents/MacOS/`) in the configuration file `config.yaml`.

### Synthetic event generation

The generation of synthetic events from the videos of lunar landing simulations is based on the video-to-event utility part of PROPHESEE's [Metavision Software Toolkit](https://www.prophesee.ai/metavision-intelligence/). Instructions detailing the installation of the open-source `OpenEB` library (sufficient for this project) and the optional, free Metavision SDK can be found [here](https://docs.prophesee.ai/stable/installation/index.html). 

## Installation

We recommend setting up a virtual environment to keep track of all the Python dependencies. Install and activate the virtual environment by running
```bash
virtualenv -p python3.9 venv
source venv/bin/activate
```

The remaining Python dependencies can be installed by running
```bash
pip install -e .
```

The following instructions will install `OpenEB` locally (taken directly from the project's [README](https://github.com/prophesee-ai/openeb/blob/main/README.md)).

### OpenEB prerequisites

Install the following dependencies:

```bash
sudo apt update
sudo apt install apt-utils build-essential software-properties-common wget unzip curl git cmake
sudo apt install libopencv-dev libboost-all-dev libusb-1.0-0-dev
sudo apt install libhdf5-dev hdf5-tools libglew-dev libglfw3-dev libcanberra-gtk-module ffmpeg 
```


The Python bindings of the C++ API rely on the [pybind11](https://github.com/pybind) library, specifically version 2.6.0. Unfortunately, there is no pre-compiled version of pybind11 available on Ubuntu, so you need to install it manually:

```bash
wget https://github.com/pybind/pybind11/archive/v2.6.0.zip
unzip v2.6.0.zip
cd pybind11-2.6.0/
mkdir build && cd build
cmake .. -DPYBIND11_TEST=OFF
cmake --build .
sudo cmake --build . --target install
cd ../../
```

### OpenEB compilation

 1. Retrieve the `OpenEB` source code:
    ```bash
    git clone https://github.com/prophesee-ai/openeb.git --branch 4.2.1
    cd openeb
    ```
 2. Create and open the build directory in the `openeb` folder (absolute path to this directory is called `OPENEB_SRC_DIR` in next sections)
    ```bash
    mkdir build && cd build
    ```
 3. Generate the makefiles using CMake. If you want to specify to cmake which version of Python to consider, you should use the option `-DPython3_EXECUTABLE`:
    ```bash
    cmake .. -DBUILD_TESTING=OFF -DPython3_EXECUTABLE=/usr/bin/python3.9 -DPYTHON3_SITE_PACKAGES=<path-to>/event-based-science/venv/lib/python3.9/site-packages/
    ```
 4. Compile:
    ```bash
    cmake --build . --config Release -- -j 4
    ```
 5. Deploy `OpenEB` in the system path
    ```bash
    sudo cmake --build . --target install
    ```
 6. Update `LD_LIBRARY_PATH` and `HDF5_PLUGIN_PATH` (which you may add to your `~/.bashrc` to make it permanent):
    ```bash
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
    export HDF5_PLUGIN_PATH=$HDF5_PLUGIN_PATH:/usr/local/hdf5/lib/plugin
    ```

    Note that you ou can also deploy the OpenEB files (applications, samples, libraries etc.) in a directory of your choice by using
    the `CMAKE_INSTALL_PREFIX` variable (`-DCMAKE_INSTALL_PREFIX=<OPENEB_INSTALL_DIR>`) when generating the makefiles
    in step 3. Similarly, you can configure the directory where the Python packages will be deployed using the
    `PYTHON3_SITE_PACKAGES` variable (`-DPYTHON3_SITE_PACKAGES=<PYTHON3_PACKAGES_INSTALL_DIR>`).


## Citing


