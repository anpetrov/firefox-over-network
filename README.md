firefox-over-network
====================

This set of patches enables Firefox to run Content Processes remotely over TCP/IP.
It allows to create light-weight FF client which only does compositing and handles user interaction,
while server runs javascript, rendering, etc. This efficiently allows creation of Cloud-based browser 
while efficiently using almost off-the-shelf FF source code.

For example, I used this approach to run client on on iPad (with a different set of patches enabling FF on iOS) 
while server part ran on EC2 linux servers.

Please note: this code is of prototype quality which was hacked together in a couple of days. it can be only
used as proof-of-concept.

Patch contents
--------------

a) SurfaceDescriptorNetwork
This is a new SurfaceDescriptor type (among SurfaceDescriptorX11, etc) which handles networking

b) OpPaintThebesBufferNet
A new OP which tries to convey changes of Content Process surfaces to the parent.

c) Rest of code which patches various aspects of Layer Manager

d) dummy sockets for chromium-ipc

How to run
----------

a) apply the patches (originally on top of b50def6f7388 in mozilla-central)

b) build 'mobile' application (--enable-application=mobile) on Linux with default backend (gtk-cairo) 

c) put binaries at the same location on BOTH client and server (e.g /tmp/firefox-net)

d) first run server:
  $ MOZ_LAYERS_FORCE_NETWORK_SURFACES=1 ./run-mozilla.sh  ./plugin-container 00000 tab
  this makes server process listen on localhost:12345
 
e) then run client fennec:
  $ MOZ_LAYERS_FORCE_NETWORK_SURFACES=1 ./run-mozilla.sh ./fennec
  this will start local fennec and make it connect to localhost:12345. You might want to change localhost to 
  your real server address or do some tunnelling
  
f) try loading some pages and scroll. It should work

Notes
-----

  * in order to save network traffic you might want to set
    browser.ui.zoom.pageFitGranularity to 1 otherwise you probably will be getting tons of useless repaints
  * the code does some work of filtering out 'bogus' repaints by calculating crc32 on repainted areas. This is 
    unreliable, disable it
  * Lossful JPEG compression is used for pixel data. The quality ratio is rather ungodly, you might want to tune it
    too.
