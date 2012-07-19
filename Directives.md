## Core
#### rtmp
syntax: `rtmp { ... }`  
context: root  
The block which holds all RTMP settings

#### server
syntax: `server { ... }`  
context: rtmp  
Declares RTMP server instance

    rtmp {
      server {
      }
    }

#### listen
syntax: `listen (addr[:port]|port|unix:path) [bind]  [ipv6only=on|off] [so_keepalive=on|off|keepidle:keepintvl:keepcnt]`  
context: server  

Adds listening socket to NGINX for accepting RTMP connections

    server {
        listen 1935;
    }

#### application
syntax: `application name { ... }`  
context: server  

Creates RTMP application. Unlike http location application name cannot
be a pattern.

    server {
        listen 1935;
        application myapp {
        }
    }

#### timeout
syntax: `timeout value`  
context: rtmp, server  

Socket timeout. This value is primarily used for writing. Most of time RTMP 
module does not expect any activity on all sockets except for publisher socket. 
If you want broken socket to get quickly disconnected use active tools like 
keepalive or RTMP ping. Default is 1 minute.

    timeout 60s;

#### ping
syntax: `ping value`  
context: rtmp, server  

RTMP ping interval. Zero turns ping off. RTMP ping is a protocol feature for
active connection check. A special packet is sent to remote peer and a reply
is expected within a timeout specified with ping_timeout directive. If ping
reply is not received within this time then connection is closed. Default 
value for ping is 0 (turned off). Default ping timeout is 30 seconds.

    ping 3m;
    ping_timeout 30s;

#### ping_timeout
syntax: `ping_timeout value`  
context: rtmp, server  

See ping description above.

#### max_streams
syntax: `max_streams value`  
context: rtmp, server  

Sets maximum number of RTMP streams. Data streams are multiplexed into
a single data stream. Different channels are used for sending commands,
audio, video etc. Default value is 32 which is usually ok for many cases.

    max_streams 32;
        
#### ack_window
syntax: `ack_window value`  
context: rtmp, server  

Sets RTMP acknowledge window size. It's the number of bytes received after
which peer should send acknowledge packet to remote side. Default value is
5000000.

    ack_window 5000000;

#### chunk_size
syntax: `chunk_size value`  
context: rtmp, server  

Maximum chunk size for stream multiplexing. Default is 4096. The bigger
this value the lower CPU overhead. This value cannot be less than 128.

    chunk_size 4096;

#### max_queue

#### max_message
syntax: `max_queue value`  
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

#### allow
Syntax: `allow [play|publish] address|subnet|all`  
Context: rtmp, server, application  

Allow publishing/playing from addresses specified or from all addresses.
Allow/deny directives are checked in order of appearance.

    allow publish 127.0.0.1;
    deny publish all;
    allow play 192.168.0.0/24;
    deny play all;

#### deny
Syntax: `deny [play|publish] address|subnet|all`  
Context: rtmp, server, application  

See allow for description.

## Exec

#### exec
Syntax: `exec command arg*`  
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
        exec /usr/bin/ffmpeg -i rtmp://localhost/src/$name -vcodec libx264 -vprofile baseline -g 10 -s 300x200 -acodec libfaac -ar 44100 -ac 1 -f flv rtmp://localhost/hls/$name;
    }

    application hls {
        hls on;
        hls_path /tmp/hls;
        hls_fragment 15s;
    }


#### respawn
Syntax: `respawn on|off`  
Context: rtmp, server, application  

If turned on respawns child process when it's terminated while publishing
is still on. Default is on;

    respawn off;

#### respawn_timeout
Syntax: `respawn_timeout timeout`  
Context: rtmp, server, application  

Sets respawn timeout to wait before starting new child instance.
Default is 5 seconds.

    respawn_timeout 10s;

## Live

#### live
Syntax: `live on|off`  
Context: rtmp, server, application  

Toggles live mode i.e. one-to-many broadcasting.

    live on;

#### meta
Syntax: `meta on|off`  
Context: rtmp, server, application  

Toggles sending metadata to clients. Defaults to on.

    meta off;

#### stream_buckets

## Record

#### record
syntax: `record [off|all|audio|video|keyframes]*`  
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
syntax: `record_path path`  
context: rtmp, server, application  

Specifies record path to put recorded flv files to.

    record_path /tmp/rec;

#### record_suffix
syntax: `record_suffix value`  
context: rtmp, server, application  

Sets record file suffix. Defaults to '.flv'.

    record_suffix _recorded.flv;

#### record_unique
syntax: `record_unique on|off`  
context: rtmp, server, application  

If turned on appends current timestamp to recorded files. Otherwise the same file
is re-written each time new recording takes place. Default is off.

    record_unique off;

#### record_max_size
syntax: `record_max_size size`  
context: rtmp, server, application  

Set maximum recorded file size.

    record_max_size 128K;

#### record_max_frames
syntax: `record_max_frames nframes`  
context: rtmp, server, application  

Sets maximum number of video frames per recorded file.

    record_max_frames 2;

#### record_interval
syntax: `record_interval time`  
context: rtmp, server, application
  
Restart recording after this number of (milli)seconds.
Off by default. Zero means no delay between recordings. If
record_unique is off then all record fragments are written to the
same file. Otherwise timestamp is appended which makes files
differ (given record_interval is longer than 1 second).

    record_interval 1s;

    record_interval 15m;

#### on_record_done

## Video on demand

#### play
Syntax: `play dir`  
Context: rtmp, server, application  

PLay FLVs from specified directory. Indexed
FLVs are played with random seek capability.
Unindexed FLVs are played with seek/pause disabled
(restart-only mode). Use FLV indexer (for example, yamdi)
for indexing.

    application vod {
        play /var/flvs;
    }

Playing /var/flvs/dir/file.flv:

    ffplay rtmp://localhost/vod/dir/file.flv

## Relay

#### pull
Syntax: `pull url [key=value]*`  
Context: application  

Creates pull relay. Stream is pulled from remote machine
and becomes available locally. It only happens when at least
one player is playing the stream locally.

Url syntax: `[rtmp://]host[:port][/app[/playpath]]`. If application
is missing then local application name is used. If playpath is missing
then current stream name is used instead.

The following parameters are supported:
* app - explicit application name
* name - local stream name to bind relay to; if empty or non-specified then
all local streams within application are pulled
* tcUrl - auto-constructed if empty
* pageUrl - page url to pretend
* swfUrl - swf url to pretend
* flashVer - flash version to pretend, default is 'LNX.11,1,102,55'
* playPath - remote play path
* live - toggles special behavior for live streaming, values: 0,1
* start - start time in seconds
* stop - stop time in seconds

If a value for a parameter contains spaces then you should use quotes around
the **WHOLE** key=value pair like this : `'pageUrl=FAKE PAGE URL'`.

    pull rtmp://cdn.example.com/main/ch?id=12563 name=channel_a;

    pull rtmp://cdn2.example.com/another/a?b=1&c=d pageUrl=http://www.example.com/video.html swfUrl=http://www.example.com/player.swf live=1;

#### push
Syntax: `push url [key=value]*`  
Context: application  

Push has the same syntax as pull. Unlike pull push directive publishes stream to remote server.

#### push_reconnect
Syntax: `push_reconnect time`  
Context: rtmp, server, application  

Timeout to wait before reconnecting pushed connection after disconnect. Default is 3 seconds.

    push_reconnect 1s;

## Notify

#### on_play
Syntax: `on_play url`  
Context: rtmp, server, application  

Sets HTTP play callback. Each time a clients issues play command
an HTTP request is issued asynchronously and command processing is
suspended until it returns result code. If HTTP 2xx code is returned
then RTMP session continues. Otherwise connection is dropped.

HTTP request receives a number of arguments. POST method is used with
application/x-www-form-urlencoded MIME type. The following arguments are
passed to caller:
* call=play
* addr - client IP address
* app - application name
* flashVer - client flash version
* swfUrl - client swf url
* tcUrl - tcUrl
* pageUrl - client page url
* name - stream name

In addition to the above mentioned items all arguments passed explicitly to 
play command are also sent with the callback. For example if stream is
accessed with the url `rtmp://localhost/app/movie?a=100&b=face&foo=bar` then
`a`, `b` & `foo` are also sent with callback.

    on_play http://example.com/my_callback;

#### on_publish
Syntax: `on_publish url`  
Context: rtmp, server, application  

The same as on_play above with the only difference that this directive sets
callback on publish command.

#### on_done
Syntax: `on_done url`  
Context: rtmp, server, application  

Sets play/publish terminate callback. All the above applies here. However
HTTP status code is not checked for this callback. This callback is repeated
on client disconnect.

## HLS

#### hls
Syntax: `hls on|off`  
Context: rtmp, server, application  

Toggles HLS on the application.

    hls on;
    hls_path /tmp/hls;
    hls_fragment 15s;

#### hls_path
Syntax: `hls_path path`  
Context: rtmp, server, application  

Sets HLS playlist and fragment directory. This directory should
exist before NGINX starts. It should be located in memory (tmpfs)
for best results.

#### hls_fragment
Syntax: `hls_fragment time`  
Context: rtmp, server, application  

Sets HLS fragment length. Defaults to 5 seconds.

#### hls_playist_length
Syntax: `hls_playlist_length time`  
Context: rtmp, server, application  

Sets HLS playlist length. Defaults to 30 seconds.

    hls_playlist_length 10m;

## Statistics

Statistics module is NGINX HTTP module unlike all other modules listed
here. Hence statistics directives should be located within http{} block.

#### rtmp_stat
Syntax: `rtmp_stat all`  
Context: http, server, location  

Sets RTMP statistics handler to the current HTTP location. RTMP statistics is
dynamic XML document. To watch this document in browser as XHTML page
use rtmp_stat_stylesheet directive.

    http {
        server {
            location /stat {
                rtmp_stat all;
                rtmp_stat_stylesheet stat.xsl;
            }
            location /stat.xsl {
                root /path/to/stat/xsl/file;
            }
        }
    }

#### rtmp_stat_stylesheet
Syntax: `rtmp_stat_stylesheet path`  
Context: http, server, location  

Adds XML stylesheet reference to statistics XML to make it viewable
in browser. See rtmp_stat description and example for more information.