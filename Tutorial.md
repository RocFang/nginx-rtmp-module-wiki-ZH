## RTMP
RTMP is a proprietary protocol developed by Adobe (Macromedia) for use
in flash player. Until 2009 it had no public specification.
A number of third-party RTMP-related products started in that period
were based on the results of reverse engineering. In 2009 
[RTMP specification](http://www.adobe.com/devnet/rtmp.html) has been 
published which made developing such applications easier. However
the spec is not full and misses significant issues concerning streaming H264.

## System requirements
The module has been tested on Linux x86-family platforms. 
However it should work on FreeBSD too.

## Building NGINX with the module
Building is pretty obvious. Just cd to nginx source directory
and configure nginx this way:

`./configure --add-module=/path/to/nginx-rtmp-module`

HLS support is provided by a separate module. To add HLS 
use the following configure line:

`./configure --add-module=/path/to/nginx-rtmp-module --add-module=/path/to/nginx-rtmp-module/hls`

Then `make` and `make install`.

## Configuration

## Simple live application

## HLS (HTTP Live Streaming)
iPhone requires H264 stream to be encoded with baseline
profile. If not follow this rule the video will be jerky.

## Choosing flash player
To watch RTMP stream in browser one should either develop
flash application for that or use one of available flash
players. The most popular players which are proved to have
no problems with the module are:

* [JWPlayer](http://www.longtailvideo.com/)
* [FlowPlayer](http://flowplayer.org/)
* [Strobe Media Playback](http://www.osmf.org/strobe_mediaplayback.html)

Old versions of JWPlayer (<=4.4) supported capturing video
from webcam. You can find that version in test/ subdirectory.
However audio is not captured by this version of player.
Recent free versions of JWPlayer have no capture capability at
all.

## Transcoding streams

## Distributed streaming

## Notifications & access control

## Statistics

## Verifying session

## Utilizing multi-core CPUs