# Intel hd graphics installation on I7 7700k

```bash
apt-get -t buster-backports install linux-image-amd64 firmware-misc-nonfree intel-microcode firmware-linux-nonfree
apt remove xserver-xorg-video-intel
```

## How to test 3D acceleration (FPS) on Ubuntu 20.04 step by step instructions
The first method is the easiest as all it takes to test 3D acceleration is to install the mesa-utils package from the standard Ubuntu repository and start the test.

To do so open up terminal and execute:
$ sudo apt install mesa-utils
Once the Mesa utils are installed run the glxgears command:
$ glxgears
For more information and all available GL extensions execute:
$ glxgears -info
Testing 3D acceleration and FPS on Ubuntu 20.04 with glxgears
Testing 3D acceleration and FPS on Ubuntu 20.04 with the glxgears command.


Navigate your browser to the Phoronix Test Suite page, and download the latest .deb package. Once you have the package, install it with the gdebi command:
$ sudo apt install gdebi-core
$ sudo gdebi phoronix-test-suite_*.deb
Once the installation is complete execute the bellow command and follow the wizard to start the basic 3D acceleration test:
$ phoronix-test-suite run unigine-heaven