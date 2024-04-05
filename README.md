A.Venugopal(major)
## Installation and usage
To use this software you will have to build the executable. First, install required dependencies:
```
sudo apt-get update
sudo apt-get install make build-essential
```
Depending on OS (eg. Ubuntu Server 20.10) installing Broadcom libraries may be also required:
```
sudo apt-get install libraspberrypi-dev
```  
After installing dependencies clone this repository and use `make` command in order to build executable:
```
git clone https://github.com/markondej/fm_transmitter
cd fm_transmitter
make
```
After a successful build you can start transmitting by executing the "fm_transmitter" program:
```
sudo ./fm_transmitter -f 100.6 acoustic_guitar_duet.wav
```
Notice:
* -f frequency - Specifies the frequency in MHz, 100.0 by default if not passed
* acoustic_guitar_duet.wav - Sample WAV file
* -b bandwidth - Specifies the bandwidth in kHz, 100 by default
* -r - Loops the playback

After transmission has begun, simply tune an FM receiver to chosen frequency, you should hear the playback.
### Raspberry Pi 4
On Raspberry Pi 4 other built-in hardware probably interfers somehow with this software making transmitting not possible on all standard FM broadcasting frequencies. In this case it is recommended to:
1. Compile executable with option to use GPIO21 instead of GPIO4 (PIN 40 on GPIO header):
```
make GPIO21=1
```
2. Changing either ARM core frequency scaling governor settings to "powersave" or changing ARM minimum and maximum core frequencies to one constant value (see: https://www.raspberrypi.org/forums/viewtopic.php?t=152692 ).
```
echo "powersave"| sudo tee /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
```
3. Using lower FM broadcasting frequencies (below 93 MHz) when transmitting.
### Use as general audio output device
In order to achieve this you should load "snd-aloop" module and stream output from loopback device to transmitter application:
```
sudo modprobe snd-aloop
arecord -D hw:1,1,0 -c 1 -d 0 -r 22050 -f S16_LE | sudo ./fm_transmitter -f 100.6 - &
```
### Microphone support

arecord -D hw:1,0 -c 1 -d 0 -r 22050 -f S16_LE | sudo ./fm_transmitter -f 100.6 -
```
In cases of a performance drop down use ```plughw:1,0``` instead of ```hw:1,0``` like this:
```
arecord -D plughw:1,0 -c 1 -d 0 -r 22050 -f S16_LE | sudo ./fm_transmitter -f 100.6 -
```

transmitt uncompressed WAV (.wav) files directly or read audio data from stdin, eg. using MP3 file:
```
sudo apt-get install sox libsox-fmt-mp3
sox example.mp3 -r 22050 -c 1 -b 16 -t wav - | sudo ./fm_transmitter -f 100.6 -
```
uncompressed WAV files are supported. If you receive the "corrupted data" error try converting the file, eg. by using SoX:
```
sudo apt-get install sox libsox-fmt-mp3
sox example.mp3 -r 22050 -c 1 -b 16 -t wav converted-example.wav
sudo ./fm_transmitter -f 100.6 converted-example.wav


