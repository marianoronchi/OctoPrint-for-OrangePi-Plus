# OctoPrint-for-OrangePi-Plus
This is WIP OctoPrint for OrangePi Plus. Goal is to have OctoPi kind of image for OrangePi plus and possibly for other Armbian supported SBC's.

This is my process so far with Waveshare 7" Capacitive touch screen and Orange Pi Plus.

First you need build latest Armbian (Full image, no desktop, mainline 4.14, default kernel config) with instructions from https://docs.armbian.com/Developer-Guide_Using-Vagrant/

You may need to make the "/Armbian/build/output" directory, before the build will succeed.

When you have got the image build, flashed and booted you have to make user **pi** and add it to sudoers. This is done easily, by the Armbian on the first start. Setup your network i.e. with nmtui. If you have a (touch)screen you have to set up the correct resolution with Armbian-config. Go to System -> Bootenv and set the correct display_mode for your screen.

## Now let's install OctoPrint!
```
sudo apt-get update
sudo apt-get -y install python-pip python-dev python-setuptools git libyaml-dev build-essential
sudo pip install virtualenv
git clone https://github.com/foosel/OctoPrint.git
cd OctoPrint
virtualenv venv
./venv/bin/pip install pip --upgrade
./venv/bin/python setup.py install
sudo usermod -a -G tty pi
sudo usermod -a -G dialout pi
cd ~
```
Now lets make OctoPrint start automatically.
```
sudo cp ~/OctoPrint/scripts/octoprint.init /etc/init.d/octoprint
sudo chmod +x /etc/init.d/octoprint
sudo cp ~/OctoPrint/scripts/octoprint.default /etc/default/octoprint
```
Edit /etc/default/octoprint with correct install path.
```
sudo nano /etc/default/octoprint
```
Remove # in front of line that reads "DAEMON=/home/pi/OctoPrint/venv/bin/octoprint"
ctrl + x to save and exit.
```
sudo update-rc.d octoprint defaults
```

Now you should reboot, to check that OctoPrint starts automatically. 
If you don't have a display or don't want to set up one. You are done. Have a nice OctoPrint!



## Setting up TouchUI and auto login.

To override this, do:

sudo systemctl edit getty@tty1
And add:

[Service]
ExecStart=
ExecStart=-/sbin/agetty -a pi --noclear %I $TERM
[…]

Now:

$ systemctl cat getty@tty1 | grep Exec
ExecStart=-/sbin/agetty --noclear %I $TERM
ExecStart=
ExecStart=-/sbin/agetty -a <USERNAME> --noclear %I $TERM
And if I do:

sudo systemctl restart getty@tty1

With web browser go to **http://"ip of your device":5000**
and make the firs time settings. If I remember correctly TouchUI won't work with the user access control enabled.
Don't forget to install TouchUI from the OctoPrint plugin manager!

Then we make user pi to auto login to console.
```
sudo nano /lib/systemd/system/getty@tty1.service.d/20-autologin.conf
```
And add the fallowing to the file.
```
[Service]
ExecStart=
ExecStart=-/sbin/agetty --autologin pi --noclear %I $TERM
```

Next we are going to make our device to boot in to ToucUI. 
We need to install some packages.
```
sudo apt-get -y install --no-install-recommends xinit xinput xserver-xorg xserver-xorg-video-fbdev x11-xserver-utils matchbox 
```

Then create /usr/share/X11/xorg.conf.d/99-fbdev.conf
```
sudo nano /usr/share/X11/xorg.conf.d/99-fbdev.conf
```
and add following lines to it
```
Section "Device"
  Identifier "touchscreen"
  Driver "fbdev"
  Option "fbdev" "/dev/fb0"
EndSection
```

Then get the modified auto star files from my git
```
git clone https://github.com/MDM63/OctoPrint-TouchUI-autostart.git ~/TouchUI-autostart/
```
and install them
```
sudo cp ~/TouchUI-autostart/touchui.init /etc/init.d/touchui
sudo chmod +x /etc/init.d/touchui
sudo cp ~/TouchUI-autostart/touchui.default /etc/default/touchui
sudo update-rc.d touchui defaults

```

##Known issues
- TouchUI load-screen freezes if enabled
- Display won't power save

##TODO
- Screensaver/power off display
- Different screen orientations
- Image to install from
- Install to internal MMC
- Scrip to build the image
- Support for other boards

Enable the Camera

sudo apt-get update
sudo apt-get upgrade
sudo modprobe gc2035
sudo modprobe vfe_v4l2

sudo nano /etc/modules
//AND Add the following lines:
gc2035
vfe_v4l2
mkdir ~/motion
chmod 777 ~/motion
sudo apt-get install motion

	
sudo nano /etc/motion/motion.conf
Modify the following keys:

target_dir: Specify the destination path of the images. For example / home / pi / motion (previously created)
stream_localhost: set the parameter to off so that Motion can be accessed from another computer on the local network
Save with CTRL+X then Y

sudo cp /etc/motion/motion.conf /etc/motion.conf

Allow the Motion daemon to start at startup
To have Motion start at system startup, you need to modify a last configuration file.

sudo nano /etc/default/motion

Start manually Motion
Now we can start Motion via its service

sudo /etc/init.d/motion start

After a modification of the configuration file, it will be necessary to restart Motion to take account of the new parameters:

sudo /etc/init.d/motion restart

