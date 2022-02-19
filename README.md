# audio-automation
Audio automation

This how-to demonstrates a way to integrate Amazon Alexa with Moode Audio (https://moodeaudio.org/) (or Volumio) by using node-red as an integration platform.

There were some other options to such integration, for example create a port forward on your router and deploy an alexa skill to use remotely those API calls but this scenario has a security risk, leaving your moode audio exposed to internet.

This is a more secure approach, not having to open any incoming ports from internet.

Assuming you already have moode audio (or Volumio) installed on a Pi with DAC and already enjoying quality music, you have to install node-red in your raspberry pi (https://nodered.org/docs/getting-started/raspberrypi).

Note that node-red default installation is without password, so you have to create one. It's best practice to run node-red as a simple user, not root, preferably with no valid shell.

Then you have to create the following flows:

![flows](https://user-images.githubusercontent.com/100039669/154813898-5ebf3ce6-1137-4bbc-8c44-d57fa7b39e66.png)

In order to create these flows you need the following in your node-red:

- amazon-echo-hub, running on port 8980 (need to be added to your node-red):
- 3 amazon-echo-device (also need to installed to node-red). Only thing you have to be careful is the name (that's how you will call them from Alexa).
- For each amazon-echo-device you need to add a function and a http request node-red modules

First function is responsible to start / stop moode audio (change 10.0.0.10 with your moode internal IP):
```
res={}
if (msg.on) {
  res.url="http://10.0.0.10/command/?cmd=play"
} else {
  res.url="http://10.0.0.10/command/?cmd=stop"
}
res.payload=""
return res;
```

Next function is for skip to next / previous song:
```
res={}
if (msg.on) {
  res.url="http://10.0.0.10/command/?cmd=next"
} else {
  res.url="http://10.0.0.10/command/?cmd=previous"
}
res.payload=""
return res;
```

And final function is for handling the volume of moode audio:
```
res={}
if (msg.on) {
  res.url="http://10.0.0.10/command/?cmd=vol.sh%20-up%2010"
} else {
  res.url="http://10.0.0.10/command/?cmd=vol.sh%20-dn%2010"
}
res.payload=""
return res;
```

