
# PiCade

PiCade is a Raspberry Pi-powered mini arcade.

## Install Raspberry Pi OS.
- Flash [Raspberry Pi OS 32-bit Lite](https://downloads.raspberrypi.com/raspios_oldstable_lite_armhf/images/raspios_oldstable_lite_armhf-2024-10-28/2024-10-22-raspios-bullseye-armhf-lite.img.xz) with Raspberry Pi Imager.
- Update Bullseye:
```diff
sudo apt-get update; sudo apt-get upgrade
```
- Install the Official PIXEL Desktop:
```diff
sudo apt install xserver-xorg raspberrypi-ui-mods
```
## Install MAME and Attract-Mode.
- Install MAME:
```diff
sudo apt install mame
```
- Create a build environment:
```diff
cd ~; mkdir develop
```
- Install "sfml-pi" and Attract-Mode dependencies:
```diff
sudo apt-get install cmake libflac-dev libogg-dev libvorbis-dev libopenal-dev libfreetype6-dev libudev-dev libjpeg-dev libudev-dev libfontconfig1-dev
```
- Download and build sfml-pi:
```diff
cd ~/develop
git clone --depth 1 https://github.com/mickelson/sfml-pi sfml-pi
mkdir sfml-pi/build; cd sfml-pi/build
cmake .. -DSFML_RPI=1 -DEGL_INCLUDE_DIR=/opt/vc/include -DEGL_LIBRARY=/opt/vc/lib/libbrcmEGL.so -DGLES_INCLUDE_DIR=/opt/vc/include -DGLES_LIBRARY=/opt/vc/lib/libbrcmGLESv2.so
sudo make install
sudo ldconfig
```
- Build FFmpeg with mmal support (hardware accelerated video decoding):
```diff
cd ~/develop
wget https://ffmpeg.org/releases/ffmpeg-4.1.tar.gz
tar -xvzf ffmpeg-4.1.tar.gz
cd ffmpeg-4.1
./configure --enable-mmal --disable-debug --enable-shared
make
sudo make install
sudo ldconfig
```
- Download and build Attract-Mode:
```diff
cd ~/develop
git clone --depth 1 https://github.com/mickelson/attract attract
cd attract
make USE_GLES=1
sudo make install USE_GLES=1
```
## Add Attract-Mode to autostart.
- Create the necessary folders:
```diff
mkdir -p ~/.config/lxsession/LXDE-pi
```
- Create the autostart file:
```diff
nano ~/.config/lxsession/LXDE-pi/autostart
```
- Add the command to start Attract-Mode:
```diff
@attract
```
- Save and close the file:
Press **CTRL+X** to exit the editor, then **Y** to confirm the changes, and **Enter** to save.
- Reboot the Raspberry Pi:
```diff
sudo reboot
```
## GPIO Pin Mapping

**Joystick** (5 pins: 1 common ground and 4 directional pins)

	•	Up: GPIO 17 (Physical Pin 11)
	•	Down: GPIO 27 (Physical Pin 13)
	•	Left: GPIO 22 (Physical Pin 15)
	•	Right: GPIO 23 (Physical Pin 16)
	•	Common Ground: Any GND pin (e.g., Physical Pin 6)

**Game Buttons** (9 buttons)

	•	Button 1: GPIO 5 (Physical Pin 29)
	•	Button 2: GPIO 6 (Physical Pin 31)
	•	Button 3: GPIO 13 (Physical Pin 33)
	•	Button 4: GPIO 19 (Physical Pin 35)
	•	Button 5: GPIO 26 (Physical Pin 37)
	•	Button 6: GPIO 12 (Physical Pin 32)
	•	Button 7: GPIO 16 (Physical Pin 36)
	•	Button 8: GPIO 20 (Physical Pin 38)
	•	Button 9: GPIO 21 (Physical Pin 40)
	•	Common Ground: Any GND pin (e.g., Physical Pin 9)

**Power-Off Button**

	•	Power-Off Button: GPIO 4 (Physical Pin 7)
	•	Common Ground: Any GND pin


![GPIO Pinout Diagram](images/GPIO-Pinout-Diagram.png)

**Configuring the Power-Off Button**

- Open a terminal on your Raspberry Pi.
- Create at the script file by running:
```diff
sudo nano /home/pi/poweroff_button.py
```
- Copy the Python code above and paste it into the file:
```diff
import RPi.GPIO as GPIO
import time
import os

# Set up GPIO mode and pin
GPIO.setmode(GPIO.BCM)
POWER_PIN = 4  # GPIO 4
GPIO.setup(POWER_PIN, GPIO.IN, pull_up_down=GPIO.PUD_UP)

def shutdown(channel):
    print("Shutdown button pressed")
    os.system("sudo shutdown -h now")

# Detect a button press (falling edge)
GPIO.add_event_detect(POWER_PIN, GPIO.FALLING, callback=shutdown, bouncetime=2000)

try:
    while True:
        time.sleep(1)
except KeyboardInterrupt:
    pass
finally:
    GPIO.cleanup()
```
- Save and exit by pressing **CTRL + X**, then **Y**, then **ENTER**.
- - To test the script manually, run:
```diff
python3 /home/pi/poweroff_button.py
```

Press the button to ensure it initiates the shutdown. If it works, you’re ready to set it up to run on startup.

- Create a sistema service to run the script on startup:
```diff
sudo nano /etc/systemd/system/poweroff_button.service
```
- Add the following configuration:
```diff
[Unit]
Description=Power-Off Button Script
After=multi-user.target

[Service]
Type=simple
ExecStart=/usr/bin/python3 /home/pi/poweroff_button.py
Restart=on-failure
User=pi

[Install]
WantedBy=multi-user.target
```
- Save and exit the file by pressing **CTRL + X**, then **Y**, then **ENTER**.
- Enable and start the service:
```diff
sudo systemctl enable poweroff_button.service
sudo systemctl start poweroff_button.service
```
- Reboot the Raspberry Pi to ensure the script runs on startup:
```diff
sudo reboot
```

## Install GPIOnext
This is a GPIO controller that is fully compatible with MAME.
```diff
cd ~
git clone https://github.com/mholgatem/GPIOnext.git
bash GPIOnext/install.sh
```
After the installer runs, you will be prompted to run the configuration tool. Just follow the command prompts to set up any controls that you want. After exiting, type 'gpionext start' to run the daemon in the background.
```diff
gpionext start
```