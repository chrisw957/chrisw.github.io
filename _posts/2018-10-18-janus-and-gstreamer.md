---
layout: post
title: Janus and Gstreamer
date: 2018-10-18
---

I've been using gstreamer for a while in embedded devices.  When the need came for streaming from gstreamer to web clients,
[Janus](https://janus.conf.meetecho.com/) seemed like a good choice.  These are my notes for getting Janus installed and working with gstreamer.

# Spin Up a Server
I think [Lightsail](https://lightsail.aws.amazon.com) is a good choice.

* Go to your Lightsail console and select "Create Instance"
* For platform, select Linux/Unix, and for blueprint, "OS only" and Ubuntu 16.04 LTS.
* For instance plan, you can select something small.  I went with the $3.50/month, 512MB, 1vCPU, 20GB SSD option.
* Once the server is running, you should be able to click on the three dots menu in the corner, and connect to it with
a terminal in a broweser window.

# Add a Public IP to your Server
* Go to your Lightsail console and click on the "Networking" tab.
* Press the "Create static IP" button
* In the "Attach to an instance" section, select the name of your instance.
* Click the Create button to complete

# Open up the Firewall
* Go to your Lightsail console, and click on the "three dots menu" in the corner, then select "Manage"
* Click the "Networking" tab
* Under the Firewall section, click the plus sign to "add another"
* Leave the application as "Custom", protocol as "TCP", and enter 8088 for port rnage.

# Install Nginx
<pre>
$ sudo apt-get install nginx
$ sudo ufw allow 'Nginx Full'
</pre>
You should be able to go to http://<<your public ip>> and see the nginx welcome page now.  
The html directory is at /var/www/html.
	
# Install Janus
* Install the dependencies:
<pre>
$ sudo apt update
$ sudo apt-get install libmicrohttpd-dev libjansson-dev \
	libssl-dev libsofia-sip-ua-dev libglib2.0-dev \
	libopus-dev libogg-dev libcurl4-openssl-dev liblua5.3-dev \
	pkg-config gengetopt libtool automake gtk-doc-tools
</pre>
* Build libsrtp from source because the version supplied by Ubuntu isn't recent enough:
<pre>
$ wget https://github.com/cisco/libsrtp/archive/v1.5.4.tar.gz
$ tar xvf ./v1.5.4.tar.gz 
$ cd libsrtp-1.5.4/
$ ./configure --prefix=/usr --enable-openssl
$ make shared_library
$ sudo make install
</pre>
* Build libnice from source:
<pre>
git clone https://gitlab.freedesktop.org/libnice/libnice
cd libnice
./autogen.sh
./configure --prefix=/usr
make && sudo make install
</pre>
* Clone the Janus git repo and build it:
<pre>
$ cd ~
$ git clone https://github.com/meetecho/janus-gateway.git
$ cd janus-gateway
$ sh autogen.sh
$ ./configure --prefix=/opt/janus
$ make
$ sudo make install
$ sudo make configs
</pre>

# Copy the Janus html
<pre>
$ cd ~/janus-gateway/html
$ sudo cp  ./ /var/www/html
</pre>
You should be able to go to http://<<your public ip>> and see the Janus website now.

# Enable Janus as a systemd service
* Create a file named /lib/systemd/system/janus.service that looks like below.
<pre>
sudo vi /lib/systemd/system/janus.service
</pre>
<pre>
[Unit]
Description=Janus WebRTC Server
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/janus -o
Restart=on-abnormal
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
</pre>
* start with:
<pre>
$ sudo systemctl start janus
</pre>
* To tell the system to start janus automatically at boot:
<pre>
$ sudo systemctl enable janus.service
</pre>

# Configure the Streaming Plugin
We need to edit the config file for the streaming plugin to accept H264 video from Gstreamer.
<pre>
$ sudo vi /opt/janus/etc/janus/janus.plugin.streaming.cfg
</pre>
Under the gstreamer-sample section, edit the file to look like this:
<pre>
[gstreamer-sample]
type = rtp
id = 1
description = H264 from gstreamer
audio = no
video = yes
videoport = 5004
videopt = 126
videortpmap = H264/90000
secret = adminpwd
</pre>
Finally, restart Janus to pickup the new cfg file:
<pre>
$ sudo systemctl restart janus
</pre>

# Install gstreamer on the server
<pre>
sudo apt-get install libgstreamer1.0-0 gstreamer1.0-plugins-base gstreamer1.0-plugins-good gstreamer1.0-plugins-bad gstreamer1.0-plugins-ugly gstreamer1.0-libav gstreamer1.0-doc gstreamer1.0-tools
</pre>






