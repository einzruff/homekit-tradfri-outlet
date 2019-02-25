# homekit-tradfri-outlet
A way that the Ikea Tradfri outlet can be connected to Apple Home/HomeKit

## Prerequisites:
* Apple Home app is installed on your iPhone or iPad and setup.
* Apple Home hub (AppleTV, HomePod, or iPad as a hub) and setup.
* Ikea Tradfri Hub is plugged in and setup. Also need an Ikea control device (described below)
* Raspberry Pi (I used a Raspberry Pi 3 B+) and 8GB+ microSD card and microSD USB adapter
* PC
* Ikea Tradfri Control Outlet (comes by itself or in a kit with an on/off switch as a control device)

At the moment, 2/24/2019, HomeKit compatibility hasn’t officially been released for the Ikea Tradfri Control Outlet. Some versions of programs may have changed since then. I am not responsible for anything that results from using this guide, it is for informational purposes only. First make sure you’ve plugged your Ikea control outlet into a power outlet and you’ve connected it to a control device (on/off switch in the $14.99 kit or the 4-way dimmer switch you can buy separately). You do this by holding the control device right next to the outlet and hold its pairing button for more than 10 seconds. Keep the button held down while the light is flashing until it is solid. Once you do this, the Ikea Tradfri app should acknowledge your outlet in its app.
Set up your Pi
1. Make sure you’ve got an 8GB or larger microSD card to write Raspbian OS to.
2. Go to https://www.raspberrypi.org/downloads/raspbian/ and download “Raspbian Stretch with desktop and recommended software” 1.9GB
3. Unzip “2018-11-13-raspbian-stretch-full.zip”
4. Install Balena Etcher and plug in your microSD card to your computer with a USB adapter/etc.  https://etcher.io/  
5. In Etcher, select the IMG file from the /2018-11-13-raspbian-stretch-full folder. Select the microSD card as your drive. Click “Flash!” and wait for it to complete. Plug the card into your Pi.

## Plug it in
[NOTE: If you want to access your Pi with SSH instead of with a monitor/TV over HDMI, you’ll need to make a blank file called SSH in the BOOT directory/drive (this is on newer versions of Raspbian). Once Raspbian sees that file it will allow you to connect to it over SSH using a program like PuTTy (port 22).] Now, plug in your Pi’s power, lan cable, HDMI, mouse/keyboard/etc. You can use a program like “Fast Resolver” to find your Pi’s IP address (look for its MAC address). Default username: pi  password: raspberry     I suggest you change the default password.

## Update Raspbian
Once in SSH (or you’ve booted into the GUI and opened a terminal), 
Update the default system packages (this can take a very long time initially, I recommend doing this via a terminal so you can see its progress. It prompted me to do the updates through the GUI and doing it that way took several hours with only a progress bar telling me what it was doing. This was connected via a LAN cable on a high speed connection too. So, I recommend doing it in a terminal the following way:
	sudo apt-get update
	sudo apt-get upgrade

## Check Versions of NodeJS and g++
According to https://github.com/nfarina/homebridge/wiki/Running-HomeBridge-on-a-Raspberry-Pi it wants us to now see if Node v4.3.2 or greater is installed and if the right version of g++ is installed. I can tell you that after updating the OS, as of 2/24/19, if you type ‘nodejs –v’ you’ll see that you have NodeJS version 8.15.0 (definitely higher than the needed 4.3.2. And if you type ‘g++ -v’ you’ll see at the bottom ‘gcc version 6.3.0..’  (definitely higher than the needed 4.9).  But if for some reason you don’t have high enough versions follow the link above to find out how to get those installed 

## Install Avahi and other Dependencies
This is required by the mdns package in HAP-NodeJS library.
	sudo apt-get install libavahi-compat-libdnssd-dev

## Install homebridge and homebridge-tradfri-plugin plugin
Install homebridge globally in the following way. If you do it without these flags it will throw permission errors.
	sudo npm install –g –unsafe-perm homebridge
Now, you’ll be able to run homebridge. But, it will mention that you don’t have any plugins installed, so instead let’s go ahead and install “homebridge-tradfi-plugin” to get our Tradfri plugin going.
	sudo npm install –g homebridge-tradfri-plugin

## Install libcoap
Ok, before we get started in editing the needed config.json file for homebridge, there’s one last dependency we need installed, and that is ‘libcoap’. These are my slightly altered instructions from https://www.npmjs.com/package/homebridge-tradfri-plugin
	sudo apt-get install libtool
	sudo apt-get install autoconf
	git clone –recursive https://github.com/obgm/libcoap.git
	cd libcoap
	git checkout dtls
	git submodule update –init –recursive
	./autogen.sh
	./configure –disable-documentation –disable-shared
	make
	sudo make install

## Make our config.json file
Now, let’s make our config.json file that tells the homebridge program how to set everything up. Make sure you find your Tradfri hub’s IP address (use Fast Resolver to find it) and its “SecurityKey” (this is shown on the bottom of the hub) and replace those in the platforms section (at “host” for the IP and “key” for the SecurityKey. Since, for this guide, we’re interested in getting the control outlet to work, we set the ignoreBulbs flag to true, because for the purpose of this guide, we’re just interested in the control outlet and not any bulbs we might already have connected to the hub. Note: Sometimes there is a common error having to do with “username” in the config.json file, if you get that, change one of the numbers in the username field and the error should go away”.  We’ll use Pico as our terminal text editor.
	cd home/pi/.homebridge
	pico config.json

Insert the following into the file, change the stuff in the platform section like we discussed, then use the keyboard shortcuts at the end to write the file to the disk and exit Pico.	
{
	“bridge”: {
		“name”: “Homebridge”,
		“username: “CC:22:3D:E3:CE:32”,
		“port”: 51826,
		“pin”: “031-45-159”
	},

	“description”: “This is our Homebridge config file.”,

	“accessories”: [
	],

	“platforms”: [
	{
		“platform”: “IkeaTradfri”,
		“name”: “Tradfri”,
		“host”: “putYourHubIPHere”,
		“key”: “putYourSecurityKeyHere”,
		“ignoreBulbs”: true
	}
	]
}
Press Ctrl+O to write the file. Press Ctrl+x to exit Pico.

## Run Homebridge and Add outlet to Home App
Now, with everything configured, you can type ‘homebridge’ in the terminal and it should start without any errors. It will show a large QR code and a HomeKit code at the bottom. Load up your Home app on your iPhone and hit the plus button at the top right then “Add Accessory”. Center the camera over the QR code or enter the code in the terminal and viola! It will add your new Pi HomeKit hub. Then after that it adds the control outlet. Make sure to set which room you want them assigned to. You can now toggle the switch on and off! Use it in scenes or turn things off/on manually, your choice.

## References:
* https://www.npmjs.com/package/homebridge-tradfri-plugin
* https://github.com/nfarina/homebridge/wiki/Running-HomeBridge-on-a-Raspberry-Pi
* https://www.raspberrypi.org/downloads/raspbian/
* https://etcher.io/  
