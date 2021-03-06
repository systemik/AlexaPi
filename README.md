# AlexaPi
 
---
 
### Contributors
 
* [Sam Machin](http://sammachin.com)
* [Lenny Shirly](http://github.com/lennysh)
* [dojones1](https://github.com/dojones1)
* [Chris Kennedy](http://ck37.com)
* [Anand](http://padfoot.in)
* [Mason Stone](https://github.com/maso27)
* [Nascent Objects](https://github.com/nascentobjects)
* [Dat Bac Do](https://github.com/bdatdo0601)

---
 
This is the code needed to Turn a Raspberry Pi into a client for Amazon's Alexa service, I have developed this against the Pi 2 but I see no reason it shouldn't run on the other models. Feedback welcome.
---
##NOTE This is a continuation of a project started at https://github.com/sammachin/AlexaPi
##EXTRA NOTE This version expects the user to be logged in as 'pi' instead of 'root', as outlined in other versions.

Added in this branch:
* Voice Recognition via snowboy.  When the word "alexa" is detected, Alexa responds with a beep and records the subsequent audio to be processed.
* Push button functionality.
* Option for the user to install shairport-sync for airplay support.
* tunein support is improved
* volume control via "set volume xx" where xx is between 1 and 10

### Requirements

You will need:
* A Raspberry Pi
* An SD Card with a fresh install of Raspbian (tested against build 2015-11-21 Jessie)
* An External Speaker with 3.5mm Jack
* A USB Sound Dongle and/or Microphone
* A normally-open pushbutton connected between GPIO 18 and GND
* (Optionally) A Dual colour LED (or 2 signle LEDs) Connected to GPIO 24 & 25


Next you need to obtain a set of credentials from Amazon to use the Alexa Voice service, login at http://developer.amazon.com and Goto Alexa then Alexa Voice Service
You need to create a new product type as a Device, for the ID use something like AlexaPi, create a new security profile and under the web settings allowed origins put http://localhost:5050 and as a return URL put http://localhost:5050/code you can also create URLs replacing localhost with the IP of your Pi  eg http://192.168.1.123:5050
Make a note of these credentials you will be asked for them during the install process

### Installation

Boot your fresh Pi and login to a command prompt as pi.

Make sure you are in /home/pi

Clone this repo to the Pi
`git clone --branch snowboy https://github.com/maso27/AlexaPi.git`
Run the setup script
`sudo ./setup.sh`

Follow instructions....

Enjoy :)

### Issues/Bugs etc.

If your alexa isn't running on startup you can check /var/log/alexa.log for errrors.

If the error is complaining about alsaaudio you may need to check the name of your soundcard input device, use 
`arecord -L` 
The device name can be set in the settings at the top of main.py 

You may need to adjust the volume and/or input gain for the microphone, you can do this with 
`alsamixer`

Once the adjustments have been made, you can save the settings using
`alsactl store`

### A note on Shairport-sync

By default, shairport-sync (the airplay client) uses port 5000.  This is no problem if everything goes as planned, but AlexaPi's authorization on sammachin's design uses port 5000 as well, and if shairport-sync is running while that happens, everything fails.

As a result, I have changed the authorization port for AlexaPi to 5050.  Note that you will have to change the settings within the developer website for this to work.

### Advanced Install

For those of you that prefer to install the code manually or tweak things here's a few pointers...

The Amazon AVS credentials are stored in a file called creds.py which is used by auth_web.py and main.py, there is an example with blank values.

The auth_web.py is a simple web server to generate the refresh token via oAuth to the amazon users account, it then appends this to creds.py and displays it on the browser.

main.py is the 'main' alexa client it simply runs on a while True loop waiting either the trigger word "Alexa," or for the button to be pressed. It then records audio and when the button is released (or 5 seconds has passed in the case of the trigger word) it posts this to the AVS service using the requests library, When the response comes back it is played back using vlc via an os system call. 

The LED's are a visual indictor of status, I used a duel Red/Green LED but you could also use separate LEDS, Red is connected to GPIO 24 and green to GPIO 25, When recording the RED LED will be lit when the file is being posted and waiting for the response both LED's are lit (or in the case of a dual R?G LED it goes Yellow) and when the response is played only the Green LED is lit. If The client gets an error back from AVS then the Red LED will flash 3 times.

The internet_on() routine is testing the connection to the Amazon auth server as I found that running the script on boot it was failing due to the network not being fully established so this will keep it retrying until it can make contact before getting the auth token.

The auth token is generated from the request_token the auth_token is then stored in a local memcache with and expiry of just under an hour to align with the validity at Amazon, if the function fails to get an access_token from memcache it will then request a new one from Amazon using the refresh token.








---
 

