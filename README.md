# audio-automation
Audio automation

This how-to demonstrates a way to integrate Amazon Alexa with Moode Audio (https://moodeaudio.org/) (or Volumio) by using node-red as an integration platform.

Moode audio is a fantastic project for all audiophiles out there - especially if you combine a Pi DAC Hat on top of you Pi and a quality Hi-Fi amplifier and speakers. But although it has a great web interface where you can manage it, there is no out of the box integration with Home automation platforms such as Amazon Alexa.

There were some other options to such integration, for example create a port forward on your router and deploy an alexa skill to use remotely those API calls but this scenario has a security risk, leaving your moode audio exposed to internet.

This is a more secure approach, not having to open any incoming ports from internet.

Assuming you already have moode audio (or Volumio) installed on a Pi with DAC and already enjoying quality music, you have to install node-red in your raspberry pi (https://nodered.org/docs/getting-started/raspberrypi).

Note that node-red default installation is without password, so you have to create one. It's best practice to run node-red as a simple user, not root, preferably with no valid shell.

Now you can just upload the flows.json file to your node-red installation (you have to change some things first i.e. IP, name etc.) or follow the following guide:

You have to create the following flows:

![flows](https://user-images.githubusercontent.com/100039669/154813898-5ebf3ce6-1137-4bbc-8c44-d57fa7b39e66.png)

In order to create these flows, you need the following in your node-red:

- amazon-echo-hub, running on port 8980 (need to installed to your node-red):
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

Almost there! Deploy your node-red flows and make sure that node-red has autostart enabled (systemctl enable nodered.service)

The only thing to do in order for node-red to be accessible from Alexa is to add the following iptable rule in order to redirect port 80 to port 8980 (in order to run node-red as a simple user, you have to use high port):
```
sudo apt-get install iptables-persistent
sudo iptables -I INPUT 1 -p tcp --dport 80 -j ACCEPT
sudo iptables -A PREROUTING -t nat -i wlan0 -p tcp --dport 80 -j REDIRECT --to-port 8980
sudo netfilter-persistent save
sudo netfilter-persistent reload
```

All set. Now you only have to discover devices from your Alexa and enjoy high quality audio automation :). Note that Alexa identifies those devices as philips royal hue hub. Just say:

"Alexa, turn on Pi audio"
"Alexa, turn on Pi Audio Volume"

or create a routine for "Alexa, next song" to "Turn on Pi Next Song"

Note: Integration with Volumio is quite similar. You only have to change tha functions URI accordingly (https://volumio.github.io/docs/API/REST_API.html)
