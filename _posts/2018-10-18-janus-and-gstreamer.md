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


