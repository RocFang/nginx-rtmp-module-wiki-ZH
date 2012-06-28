## Core
#### rtmp
syntax: rtmp { ... }  
context: root  
The block which holds all RTMP settings

#### server
syntax: server { ... }  
context: rtmp  
Declares RTMP server instance

    rtmp {
      server {
      }
    }

#### listen
syntax: listen (addr[:port]|port|unix:path) [bind]  [ipv6only=on|off] [so_keepalive=on|off|keepidle:keepintvl:keepcnt]  
context: server  

Adds listening socket to NGINX for accepting RTMP connections

    server {
        listen 1935;
    }

#### application
syntax: application name { ... }  
context: server  

Creates RTMP application. Unlike http location application name cannot
be a pattern.

    server {
        listen 1935;
        application myapp {
        }
    }

#### timeout
syntax: timeout value  
context: rtmp, server  

Socket timeout. This value is primarily used for writing. Most of time RTMP 
module does not expect any activity on all sockets except for publisher socket. 
If you want broken socket to get quickly disconnected use active tools like 
keepalive or RTMP ping. Default is 1 minute.

    timeout 60s;

#### ping
syntax: ping value  
context: rtmp, server  

RTMP ping interval. Zero turns ping off. RTMP ping is a protocol feature for
active connection check. A special packet is sent to remote peer and a reply
is expected within a timeout specified with ping_timeout directive. If ping
reply is not received within this time then connection is closed. Default 
value for ping is 0 (turned off). Default ping timeout is 30 seconds.

    ping 1m;          # ping every 1 minute
    ping_timeout 30s; # wait 30 sec for ping reply

#### ping_timeout
syntax: ping_timeout value  
context: rtmp, server  

See ping description above.

#### max_streams
syntax: max_streams value  
context: rtmp, server  

Sets maximum number of RTMP streams. Data streams are multiplexed into
a single data stream. Different channels are used for sending commands,
audio, video etc. Default value is 32 which is usually ok for many cases.

    max_streams 32;
        
#### ack_window
syntax: ack_window value  
context: rtmp, server  

Sets RTMP acknowledge window size. It's the number of bytes received after
which peer should send acknowledge packet to remote side. Default value is
5000000.

    ack_window 5000000;

#### chunk_size
syntax: chunk_size value  
context: rtmp, server  

Maximum chunk size for stream multiplexing. Default is 4096. The bigger
this value the lower CPU overhead. This value cannot be less than 128.

    chunk_size 4096;

#### max_queue

#### max_message
syntax: max_queue value  
context: rtmp, server  

Maximum size of input data message. All input data comes split into
messages (and further in chunks). A partial message is kept in memory while
waiting for it to complete. In theory incoming message can be
very large which can be a problem for server stability. Default value
1M is enough for many cases.

    max_message 1M;

#### out_queue

#### out_cork

## Access

allow

deny

## Exec

#### exec
Syntax: exec command arg*  
Context: rtmp, server, application

Specifies external command with arguments to be executed on
every stream published. When publishing stops the process
is terminated. Full path to binary should be specified as the
first argument. There are no assumptions about what this process should
do. However this feature is useful with ffmpeg for stream
transcoding. FFmpeg is supposed to connect to nginx-rtmp as a client 
and output transcoded stream back to nginx-rtmp as publisher. Substitutions
can be used within command line:
* $name, ${name} - stream name
* $app, ${app} - application name

The following ffmpeg call transcodes incoming stream to HLS-ready
stream (H264/AAC). FFmpeg should be compiled with libx264 & libfaac support
for this example to work.

    application src {
        live on;
        exec /usr/bin/ffmpeg -i rtmp://localhost/app/$name -vcodec libx264 -vprofile baseline -g 10 -s 300x200 -acodec libfaac -ar 44100 -ac 1 -f flv rtmp://localhost/hls/$name;
    }

    application hls {
        hls on;
        hls_path /tmp/hls;
        hls_fragment 15s;
    }


#### respawn
Syntax: respawn on|off  
Context: rtmp, server, application  

If turned on respawns child process when it's terminated while publishing
is still on. Default is on;

    respawn off;

#### respawn_timeout
Syntax: respawn_timeout timeout
Context: rtmp, server, application

Sets respawn timeout to wait before executing new child instance.
Default is 5 seconds.

    respan_timeout 10s;

## Live

#### live
Syntax: live on|off  
Context: rtmp, server, application  

Toggles live mode i.e. one-to-many broadcasting.

    live on;

#### stream_buckets

## Record

#### record
syntax: record [off|all|audio|video|keyframes]*  
context: rtmp, server, application  

Toggles record mode. Stream can be recorded in flv file. This directive
specifies what exactly should be recorded:
* off - no recording at all
* all - audio & video (everything)
* audio - audio
* video - video
* keyframes - only key video frames

There can be any compatible combination of keys in a single record directive.


    record all;

    record audio keyframes;

#### record_path
syntax: record_path path  
context: rtmp, server, application  

Specifies record path to put recorded flv files to.

    record_path /tmp/rec;

#### record_suffix
syntax: record_suffix value    
context: rtmp, server, application  

Sets record file suffix. Defaults to '.flv'.

    record_suffix _recorded.flv;

#### record_unique
syntax: record_unique on|off  
context: rtmp, server, application  

If turned on appends current timestamp to recorded files. Otherwise the same file
is re-written each time new recording takes place. Default is off.

    record_unique off;

#### record_max_size
syntax: record_max_size size  
context: rtmp, server, application  

Set maximum recorded file size.

    record_max_size 128K;

#### record_max_frames
syntax: record_max_frames nframes  
context: rtmp, server, application  

Sets maximum number of video frames per recorded file.

    record_max_frames 2;

#### record_interval
syntax: record_interval time  
context: rtmp, server. application
  
Restart recording after this number of (milli)seconds.
Off by default. Zero means no delay between recordings. If
record_unique is off then all record fragments are written to the
same file. Otherwise timestamp is appended which makes files
differ (given record_interval is longer than 1 second).

    record_interval 1s;

    record_interval 15m;

#### on_record_done

## Relay

push

pull

relay_buffer

## Notify

#### on_play

#### on_publish

#### on_done

## Statistics

rtmp_stat

rtmp_stat_stylesheet

## HLS

#### hlh
#### hls_path
#### hls_fragment