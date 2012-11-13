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
of form $var/${var} can be used within command line:
* $name - stream name
* $app - application name
* $addr - client address
* $flashver - client flash version
* $swfurl - client swf url
* $tcurl - client tc url
* $pageurl - client page url

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

#### exec_kill_signal
Syntax: `exec_kill_signal signal`  
Context: rtmp, server, application  

Sets process termination signal. Defalt is kill (SIGKILL).
You can specify numeric or symbolic name (for POSIX.1-1990 signals).

    exec_kill_signal term;
    exec_kill_signal usr1;
    exec_kill_signal 3;

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

#### exec_publish
Syntax: `exec_publish command arg*`  
Context: rtmp, server, application

Specifies external command with arguments to be executed on
publish event. Return code is not analyzed.

#### exec_play
Syntax: `exec_play command arg*`  
Context: rtmp, server, application

Specifies external command with arguments to be executed on
play event. Return code is not analyzed.

#### exec_play_done
Syntax: `exec_play_done command arg*`  
Context: rtmp, server, application

Specifies external command with arguments to be executed on
play_done event. Return code is not analyzed.

#### exec_publish_done
Syntax: `exec_publish_done command arg*`  
Context: rtmp, server, application

Specifies external command with arguments to be executed on
publish_done event. Return code is not analyzed.

#### exec_record_done
Syntax: `exec_record_done command arg*`  
Context: rtmp, server, application, recorder

Specifies external command with arguments to be executed when
recording is finished. Additional variable `path` is supported 
for this command.

    # track client info
    exec_play bash -c "echo $addr $pageurl >> /tmp/clients";
    exec_publish bash -c "echo $addr $flashver >> /tmp/publishers";

    # convert recorded file to mp4 format
    exec_record_done ffmpeg -y -i $path -acodec libmp3lame -ar 44100 -ac 1 -vcodec libx264 $path.mp4;

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
syntax: `record [off|all|audio|video|keyframes|manual]*`  
context: rtmp, server, application  

Toggles record mode. Stream can be recorded in flv file. This directive
specifies what exactly should be recorded:
* off - no recording at all
* all - audio & video (everything)
* audio - audio
* video - video
* keyframes - only key video frames
* manual - never start recorder automatically, use control interface to start/stop

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

#### recorder
syntax: `recorder name {...}`  
context: application
  
Create recorder block. Multiple recorders can be created withing
single application. All the above mentioned recording-related 
directives can be specified in `recorder{}` block. All settings
are inherited from higher levels.

    application {
        live on;

        # default recorder
        record all;
        record_path /var/rec;

        recorder audio {
            record audio;
            record_suffix .audio.flv;
        }

        recorder chunked {
            record all;
            record_interval 15s;
            record_path /var/rec/chunked;
        }
    }

#### record_notify
syntax: `record_notify on|off`  
context: rtmp, server, application, recorder  
  
Toggles sending NetStream.Record.Start and NetStream.Record.Stop
status messages (onStatus) to publisher when specific recorder 
starts or stops recording file. Status description field holds
recorder name (empty for default recorder). Off by default.

    recorder myrec {
        record all manual;
        record_path /var/rec;
        record_notify on;
    }

## Video on demand

#### play
Syntax: `play dir|httploc`  
Context: rtmp, server, application  

PLay flv or mp4 file from specified directory or HTTP location.
If the argument is prefixed with `http://` then it is assumed
that file should be downloaded from remote http location before
playing. Note playing is not started until the whole file is
downloaded. You can use local nginx to cache files on local machine.

Indexed FLVs are played with random seek capability.
Unindexed FLVs are played with seek/pause disabled
(restart-only mode). Use FLV indexer (for example, yamdi)
for indexing.

Mp4 files can only be played if both video and audio codec are supported
by RTMP. The most common case is H264/AAC.

    application vod {
        play /var/flvs;
    }

    application vod_http {
        play http://myserver.com/vod;
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
HTTP status code is not checked for this callback.

#### on_play_done
Syntax: `on_publish_done url`  
Context: rtmp, server, application  

Same behavior as `on_done` but only for play end event.

#### on_publish_done
Syntax: `on_publish_done url`  
Context: rtmp, server, application  

Same behavior as `on_done` but only for publish end event.

#### on_record_done
syntax: `on_record_done url`  
context: rtmp, server, application, recorder  
  
Set record_done callback. In addition to common HTTP callback
variables it receives recorded file path.

    on_record_done http://example.com/recorded;

#### notify_method
syntax: `notify_method get|post`  
context: rtmp, server, application, recorder  

Sets HTTP method for notifications. Default is POST with 
`application/x-www-form-urlencoded` content type. In certain cases
GET is preferable, for example if you plan to handle the call
in `http{}` section of nginx. In this case you can use `arg_*` variables
to access arguments.

    notify_method get;

With GET method handling notifications in `http{}` section can be done this way

    location /on_play {
        if ($arg_pageUrl ~* localhost) {
            return 200;
        }
        return 500;
    }

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

## Multi-worker live streaming

Multi-worker live streaming is implemented through pushing stream
to remaining nginx workers.

#### rtmp_auto_push
Syntax: `rtmp_auto_push on|off`  
Context: root  

Toggles auto-push (multi-worker live streaming) mode.
Default is off.

#### rtmp_auto_push_reconnect
Syntax: `rtmp_auto_push_reconnect timeout`  
Context: root  

Sets auto-push reconnect timeout when worker is killed.
Default is 100 milliseconds.

#### rtmp_socket_dir
Syntax: `rtmp_socket_dir dir`  
Context: root  

Sets directory for UNIX domains sockets used for stream pushing.
Default is `/tmp`.

    rtmp_auto_push on;
    rtmp_auto_push_reconnect 1s;
    rtmp_socket_dir /var/sock;

    rtmp {
        server {
            listen 1935;
            application myapp {
                live on;
            }
        }
    }

## Control

Control module is NGINX HTTP module and should be located within http{} block.

#### rtmp_control
Syntax: `rtmp_control all`  
Context: http, server, location  

Sets RTMP control handler to the current HTTP location. 

    http {
        server {
            location /control {
                rtmp_control all;
            }
        }
    }

[More details about control module](Control-module)