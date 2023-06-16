## Compiling the NOAA ARL era52arl program on Linux

A step-by-step guide for compiling the ECMWF eccodes libraries and ARL era52arl program on Linux.

To use ECMWF ERA-5 reanalysis as the input meteorological data for the HYSPLIT model, the conversion program 'era52arl' must be compiled from source by the user. I am sure this is trivial for folks experienced with computers, but for me this was an uphill battle. I wrote this guide as I worked through the various steps and problems I encountered. It is not perfect or universal - you may encounter steps that are incompatible with your system, some steps may be unnecessary, and so forth. I hope it's useful. 

It is theoretically possible to do all of this on Windows but I do not recommend it, as I found that parts of the eccodes libraries were written in a way that is incompatible with windows (see e.g. https://github.com/ecmwf/eccodes-python/issues/35). If you are absolutely set on compiling on Windows, I have a lengthy document detailing my attempts at compiling the eccodes libraries (I was not able to successfully create the eccodes binaries in Cmake using either UNIX Makefile provided by MSYS2 or with the Microsoft Visual route incl. intel compilers) that may be of use to you (see 'Windows attempts' folder in this repo). I strongly suggest installing linux (it is quite straightforward and the process does not really vary between Windows OS to my knowledge: https://www.tomshardware.com/how-to/dual-boot-linux-and-windows-11). 

### Notes
- Commands to be written in the terminal are bounded by ` `, but should be copied without the bounding punctuation.
- The `sudo` command prefix will execute an action as the root user, bypassing typical ‘permission denied’ problems. This may or may not be required depending on where you have decided to compile/install/make the various programs in this guide.
- To copy a folder and all its contents use `sudo cp -p -f -r /path/to/source/* /path/to/dest`

Created using Linux Mint Cinnamon 21.1 (64-bit). Adjustments may be needed for other linux distributions.

1. Get the eccodes tarball from https://confluence.ecmwf.int/display/ECC/Releases.
      - You’re after the most recent release’s tar
2. Get cmake
      - Open the terminal (ctrl + alt + T)
      - Enter `sudo apt install cmake-qt-gui`
      - To confirm the install was successful, run `cmake-gui` in the terminal
      - Cmake should launch.
      - From the command line, run `cmake --help`. A series of options for the cmake program should appear
3. Install gfortran
      - In terminal: `sudo apt install gfortran`
      - To confirm the installation was successful, run `which gfortran`. The install directory should be returned.
4. Install AEC from https://gitlab.dkrz.de/k202009/libaec
      - Get most recent tarball
      - Open terminal from inside the downloads folder
      - Extract the tarball: `tar -xzf libaec-v1.0.6.tar.gz`
      - Close terminal. Open a new one from inside the root directory
      - Create or cd into the opt folder:
           - `cd /opt` or 
           - `sudo mkdir /opt` then `cd /opt`
      - Copy tarball to the /opt folder: `sudo cp -r ~/Downloads/libaec-v1.0.6 /opt/libaec-v1.0.6`
      - Make a build directory: `sudo mkdir build`
      - Cd into build directory: `cd build`
      - Use cmake to make the build:
           - `sudo cmake -S /opt/libaec-v1.0.6 -D CMAKE_INSTALL_PREFIX=/opt/libaec`
      - Install: `sudo make install`
      - Clean up; delete both the source and binaries from the /opt folder using e.g. `sudo rm -r /opt/build`
5. Insall the OpenJPG library: https://github.com/uclouvain/openjpeg. Note: this may or may not be necessary; I couldn’t figure out if cmake was actually using the openjpeg library I specified. 
      - As with AEC, the method here uses cmake.
      - On the github page, click on code -> download .zip
      - Extract the archive inside the Downloads folder (this can either be done in the GUI or using e.g. unzip).
      - Cd back to the /opt directory if you left it: 
           - `cd /opt`
      - Make and cd into a build directory:
           - `sudo mkdir build`
           - `cd build`
      - Copy the openjpeg-master folder to the /opt folder
           - `sudo cp -r ~/Downloads/openjpeg-master /opt/openjpeg-master
      - Use cmake to make the build:
           - `sudo cmake -S /opt/openjpeg-master -D CMAKE_INSTALL_PREFIX=/opt/openjpeg`
      - Then install:
            - `sudo make install`
      - Clean up
           - `sudo rm -r /opt/build`
           - `sudo rm -r /opt/openjpeg-master`
6. Install lipng
      - On linux, this can be done with `sudo apt-get install libpng-dev`
      - Note the install directory returned by `whereis libpng`
7. Install netcdf. 
      - `sudo apt-get install libnetcdf-dev`
8. Install git.
      - `sudo apt-get install git`
9. Install miniconda and numpy.
      - Follow https://conda.io/projects/conda/en/stable/user-guide/install/linux.html.
      - Download the latest .sh file for linux
      - Open the terminal in the downloads folder
           - `cd ~/Downloads`
      - Run the .sh installer
           - `bash Miniconda3-latest-Linux-x86_64.sh`
      - Note that ‘latest’ refers to whatever version you downloaded. Make sure the actual file name matches what you downloaded.
      - Follow the prompts.
      - When miniconda is installed, run `conda list`. You should get a list of packages.
      - Next, install numpy
           - `conda install numpy`
10. Install the eccodes library.
      - Download the most recent tarball from https://confluence.ecmwf.int/display/ECC/Releases.
      - As before, unzip the file inside the Downloads directory
           - `cd ~/Downloads`
           - `tar -zxf eccodes-2.30.2-Source.tar.gz`
      - Change to the /opt folder
           - `cd /opt`
      - Copy the unpacked source folder from the Downloads folder to /opt
           - `sudo cp -r ~/Downloads/eccodes-2.30.2-Source /opt/eccodes-2.30.2-Source`
      - Create and cd into a build folder
           - `sudo mkdir build`
           - `cd build`
      - Create the binaries with cmake, supplying several variables that match the various libraries downloaded (AEC, openjpeg and libpng). This step can be done optionally in the GUI. 
      - I ultimately decided to install into /opt to fit with the implied defaults of the era52arl program as discussed in the hysplit forums. Another possible location is /usr/local
      - Optional cmake flags are specified with ‘-D’. Here are the various flags to set, based on my own install on linux mint:
           - Source location: -S /usr/local/eccodes-2.30.2-Source
           - Install prefix: -D CMAKE_INSTALL_PREFIX=/usr/local//eccodes-2.30.2
           - AEC location: -D AEC_DIR=/opt/libaec
           - Enable openjpeg: -D ENABLE_JPEG_LIBOPENJPEG=ON
           - Specify openjpeg location: -D OPENJPEG_DIR=/opt/openjpeg
      - Here’s the full line of code I used: `sudo cmake -S /opt/eccodes-2.30.2-Source -D CMAKE_INSTALL_PREFIX=/opt/eccodes -D AEC_DIR=/opt/libaec -D OPENJPEG_DIR=/opt/openjpeg`
      - Next, make the build: `sudo make`
      - Test: ‘sudo ctest’
      - Then, install: `sudo make install`
      - Next, delete the build and source directories
           - `cd ..`
           - `sudo rm -r build`
           - `sudo rm -r eccodes-2.30-2-Source`
      - Eccodes should now be installed.
11. Obtain the data2arl library from ARL and make the libhysplit.a library.
      - HYSPLIT data2arl distro from https://www.ready.noaa.gov/HYSPLIT_data2arl.php
      - Download the ‘https://www.ready.noaa.gov/data/web/models/hysplit4/decoders/hysplit_data2arl.zip’. I used the June 13, 2022 version.
      - This is also included as standard in recent HYSPLIT distributions (look for the ‘data2arl’ folder)
      - Here, I assume the folder is called ‘hysplit_data2arl’ when unzipped inside the /opt folder
      - Within the data2arl folder, there are bundled conversion utilities, including era52arl. 
      - The project called ‘metprog’ contains all the data needed for the libhysplit
      - Open the terminal within the metprog folder.
           - If hysplit_data2arl is in the /opt/ folder, `cd /opt/hysplit_data2arl/metprog/library`
      - Modify the makefile to suit your system.
           - Open the Makefile inside /opt/hysplit_data2arl/metprog/library inside a suitable editor. I used Visual Studio Code. 
           - If the folder structure is left as a default, you just need to direct this makefile to the ‘include’ makefile inside the first data2arl directory. 
           - To do this, change the line:
               - include ../../Makefile.inc
               - To:
               - include /opt/hysplit_data2arl/Makefile.inc.gfortran
      - Save the Makefile
      - Make the libhysplit.a library: `sudo make`
      - This should create a file called libhysplit.a inside the library folder.
12. Make the era52arl program.
      - This requires modifying a pair of makefiles.
      - The first should be in the main hysplit_data2arl folder, called ‘Makefile.inc.gfortran’
           - Various libraries are linked. Make sure the paths reflect real files on your system.
      - Next, there’s the makefile inside the era52arl directory.
           - First, make sure the Makefile.inc.gfortran is included. 
           - To do this, change the line:
               - include ../../Makefile.inc
               - To:
               - include /opt/hysplit_data2arl/Makefile.inc.gfortran
           - Make sure the previously compiled hysplit library is linked. Specify ‘LIBHYS’ at that line:
               - For me, LIBHYS = -L ../metprog/library -lhysplit
      - At this point you can attempt to compile the program by running `sudo make` inside the era52arl folder.
      - If it is successful, run `./era52arl` inside the era52arl folder without any arguments to test the program.
      - If you receive the following error, you can try adding symlinks to the eccodes libraries inside your usr/lib/ folder. 
> ./era52arl: error while loading shared libraries: libeccodes_f90.so: cannot open shared object file: No such file or directory

      - To add the libraries to the usr/lib/ folder, use these lines in terminal. The syntax is `sudo ln -s [TARGET] [LINK]`
           - `sudo ln -s /opt/eccodes/lib/libeccodes_f90.so /usr/lib/libeccodes_f90.so`
           - `sudo ln -s /opt/eccodes/lib/libeccodes.so /usr/lib/libeccodes.so`

At this point, I was able to successfully run the era52arl program. 
