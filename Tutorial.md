## RTMP

## System requirements

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
from webcam. You can see that version in test/ subdirectory.
However audio is not captured by this version of player.
Recent free versions of JWPlayer have no capture capability at
all.

## Transcoding streams

## Distributed streaming

## Notifications & access control

## Statistics

## Verifying session

## Utilizing multi-core CPUs