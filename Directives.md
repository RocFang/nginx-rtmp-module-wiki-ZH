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
value for ping is 1 minute. Default ping timeout is 30 seconds.

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

Shell-style redirects can be specified in exec directive for writing output
and accepting input. Supported are
* truncating output `>file`
* appending output `>>file`
* descriptor redirects like `1>&2`
* input `<file`

The following ffmpeg call transcodes incoming stream to HLS-ready
stream (H264/AAC). FFmpeg should be compiled with libx264 & libfaac support
for this example to work.

    application src {
        live on;
        exec ffmpeg -i rtmp://localhost/src/$name -vcodec libx264 -vprofile baseline -g 10 -s 300x200 -acodec libfaac -ar 44100 -ac 1 -f flv rtmp://localhost/hls/$name 2>>/var/log/ffmpeg-$name.log;
    }

    application hls {
        live on;
        hls on;
        hls_path /tmp/hls;
        hls_fragment 15s;
    }

#### exec_static
Syntax: `exec_static command arg*`  
Context: rtmp, server, application

Similar to `exec` but runs specified command at nginx start.
Does not support substitutions since has no session context.

    exec_static ffmpeg -i http://example.com/video.ts -c copy -f flv rtmp://localhost/myapp/mystream;

#### exec_kill_signal
Syntax: `exec_kill_signal signal`  
Context: rtmp, server, application  

Sets process termination signal. Default is kill (SIGKILL).
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
publish event. Return code is not analyzed. Substitutions of `exec`
are supported here as well. In addition `args` variable is supported
holding query string arguments.

#### exec_play
Syntax: `exec_play command arg*`  
Context: rtmp, server, application

Specifies external command with arguments to be executed on
play event. Return code is not analyzed. Substitution list
is the same as for `exec_publish`.

#### exec_play_done
Syntax: `exec_play_done command arg*`  
Context: rtmp, server, application

Specifies external command with arguments to be executed on
play_done event. Return code is not analyzed. Substitution list
is the same as for `exec_publish`.

#### exec_publish_done
Syntax: `exec_publish_done command arg*`  
Context: rtmp, server, application

Specifies external command with arguments to be executed on
publish_done event. Return code is not analyzed. Substitution list
is the same as for `exec_publish`.

#### exec_record_done
Syntax: `exec_record_done command arg*`  
Context: rtmp, server, application, recorder

Specifies external command with arguments to be executed when
recording is finished. Substitution of `exec_publish` are supported here
as well as additional variables `path` and `recorder`.

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

#### interleave
Syntax: `interleave on|off`  
Context: rtmp, server, application  

Toggles interleave mode. In this mode audio and video
data is transmitted on the same RTMP chunk stream.
Defaults to off.

    interleave on;

#### wait_key
Syntax: `wait_key on|off`  
Context: rtmp, server, application  

Makes video stream start with a key frame. Defaults to off.

    wait_key on;

#### wait_video
Syntax: `wait_video on|off`  
Context: rtmp, server, application  

Disable audio until first video frame is sent. Defaults to off.
Can be combined with `wait_key` to make client receive video
key frame with all other data following it. However this usually
increases connection delay. You can tune keyframe interval in your
encoder to reduce the delay.

    wait_video on;

#### publish_notify
Syntax: `publish_notify on|off`  
Context: rtmp, server, application  

Send `NetStream.Publish.Start` and `NetStream.Publish.Stop` to
subscribers. Defaults to off.

    publish_notify on;

#### drop_idle_publisher
Syntax: `drop_idle_publisher timeout`  
Context: rtmp, server, application  

Drop publisher connection which has been idle (no audio/video data)
within specified time. Default is off. Note this only works when
connection is in publish mode (after sending `publish` command).

    drop_idle_publisher 10s;

#### sync
Syntax: `sync timeout`  
Context: rtmp, server, application  

Synchronize audio and video streams. If subscriber bandwidth
is not enough to receive data at ublisher rate some frames are
dropped by server. This leads to synchronization problem. When 
timestamp difference exceeds the value specified as `sync` argument an 
absolute frame is sent fixing that. Default is 300ms.

    sync 10ms;

#### play_restart
Syntax: `play_restart on|off`  
Context: rtmp, server, application  

If enabled nginx-rtmp sends NetStream.Play.Start and NetStream.Play.Stop
to each subscriber every time publisher starts or stops publishing. If disabled
each subscriber receives those notifications only at the start and end of
playback. Default is on.

    play_restart off;

## Record

#### record
syntax: `record [off|all|audio|video|keyframes|manual]*`  
context: rtmp, server, application, recorder  

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
context: rtmp, server, application, recorder  

Specifies record path to put recorded flv files to.

    record_path /tmp/rec;

#### record_suffix
syntax: `record_suffix value`  
context: rtmp, server, application, recorder  

Sets record file suffix. Defaults to '.flv'.

    record_suffix _recorded.flv;

Record suffix can be a pattern in `strftime` format.
The following directive

    record_suffix -%d-%b-%y-%T.flv

will produce files of the form `mystream-24-Apr-13-18:23:38.flv`.
All supported `strftime` format options can be found on 
[strftime man page](http://pubs.opengroup.org/onlinepubs/009695399/functions/strftime.html).

#### record_unique
syntax: `record_unique on|off`  
context: rtmp, server, application, recorder  

If turned on appends current timestamp to recorded files. Otherwise the same file
is re-written each time new recording takes place. Default is off.

    record_unique on;

#### record_append
syntax: `record_append on|off`  
context: rtmp, server, application, recorder  

Toggles file append mode. When turned on recorder appends new data to the old file
or creates it when it's missing. There's no time gap between the old data and the new
data in file. Default is off.

    record_append on;

#### record_lock
syntax: `record_lock on|off`  
context: rtmp, server, application, recorder  

When turned on currently recorded file gets locked with `fcntl` call.
That can be checked from elsewhere to find out which file is being recorded.
Default is off.

    record_lock on;

On FreeBSD you can use `flock` tool to check that. On Linux `flock` and `fcntl`
are unrelated so you are left with writing a simple script checking file lock status.
Here's an example of such script `isunlocked.py`.

    #!/usr/bin/python

    import fcntl, sys

    sys.stderr.close()
    fcntl.lockf(open(sys.argv[1], "a"), fcntl.LOCK_EX|fcntl.LOCK_NB)

#### record_max_size
syntax: `record_max_size size`  
context: rtmp, server, application, recorder  

Set maximum recorded file size.

    record_max_size 128K;

#### record_max_frames
syntax: `record_max_frames nframes`  
context: rtmp, server, application, recorder  

Sets maximum number of video frames per recorded file.

    record_max_frames 2;

#### record_interval
syntax: `record_interval time`  
context: rtmp, server, application, recorder
  
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
Syntax: `play dir|http://loc [dir|http://loc]*`  
Context: rtmp, server, application  

PLay flv or mp4 file from specified directory or HTTP location.
If the argument is prefixed with `http://` then it is assumed
that file should be downloaded from remote http location before
playing. Note playing is not started until the whole file is
downloaded. You can use local nginx to cache files on local machine.

Multiple play locations can be specified in a single `play` directive.
When multiple `play` directives are specified the location lists
are merged and inherited from higher scopes. An attempt to play
each location is made until a successful location is found.
If such location is not found error status is sent to client.

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

    application vod_mirror {
        # try local location first, then access remote location
        play /var/local_mirror http://myserver.com/vod;
    }

Playing /var/flvs/dir/file.flv:

    ffplay rtmp://localhost/vod//dir/file.flv

The two slashes after `vod` make ffplay use `vod` and application name
and the rest of the url as playpath.

#### play_temp_path
Syntax: `play_temp_path dir`  
Context: rtmp, server, application  

Sets location where remote VOD files are stored before playing.
Default is `/tmp`;

    play_temp_path /www;
    play http://example.com/videos;

#### play_local_path
Syntax: `play_local_path dir`  
Context: rtmp, server, application  

Sets location where remote VOD files copied from `play_temp_path`
directory after they are completely downloaded. Empty value
disables the feature. By default it's empty. The feature can be used
for caching remote files locally. 

This path should be on the same device as `play_temp_path`.

    # search file in /tmp/videos.
    # if not found play from remote location
    # and store in /tmp/videos

    play_local_path /tmp/videos;
    play /tmp/videos http://example.com/videos;

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
* static - makes pull static, such pull is created at nginx start

If a value for a parameter contains spaces then you should use quotes around
the **WHOLE** key=value pair like this : `'pageUrl=FAKE PAGE URL'`.

    pull rtmp://cdn.example.com/main/ch?id=12563 name=channel_a;

    pull rtmp://cdn2.example.com/another/a?b=1&c=d pageUrl=http://www.example.com/video.html swfUrl=http://www.example.com/player.swf live=1;

    pull rtmp://cdn.example.com/main/ch?id=12563 name=channel_a static;

#### push
Syntax: `push url [key=value]*`  
Context: application  

Push has the same syntax as pull. Unlike pull push directive publishes stream to remote server.

#### push_reconnect
Syntax: `push_reconnect time`  
Context: rtmp, server, application  

Timeout to wait before reconnecting pushed connection after disconnect. Default is 3 seconds.

    push_reconnect 1s;

#### session_relay
Syntax: `session_relay on|off`  
Context: rtmp, server, application  

Toggles session relay mode. In this mode relay is destroyed when connection is closed.
When the setting is off relay is destroyed when stream is closed so that another relay
could possibly be created later. Default is off.

    session_relay on;

## Notify

#### on_connect
Syntax: `on_connect url`  
Context: rtmp, server  

Sets HTTP connection callback. When clients issues connect command
an HTTP request is issued asynchronously and command processing is
suspended until it returns result code. If HTTP 2xx code is returned
then RTMP session continues. The code of 3xx makes RTMP redirect
to another application whose name is taken from `Location` HTTP
response header. Otherwise connection is dropped.

Note this directive is not allowed in application scope since
application is still unknown at connection stage.

HTTP request receives a number of arguments. POST method is used with
application/x-www-form-urlencoded MIME type. The following arguments are
passed to caller:
* call=connect
* addr - client IP address
* app - application name
* flashVer - client flash version
* swfUrl - client swf url
* tcUrl - tcUrl
* pageUrl - client page url

In addition to the above mentioned items all arguments passed explicitly to 
connect command are also sent with the callback. You should distinguish
connect arguments from play/publish arguments. Players usually have a special
way of setting connection string separate from play/publish stream name.
As an example here's how these arguments are set in JWPlayer

    ...
    streamer: "rtmp://localhost/myapp?connarg1=a&connarg2=b",
    file: "mystream?strarg1=c&strarg2=d",
    ...

Ffplay (with librtmp) example

    ffplay "rtmp://localhost app=myapp?connarg1=a&connarg2=b playpath=mystream?strarg1=c&strarg2=d"

Usage example

    on_connect http://example.com/my_auth;

Redirect example

    location /on_connect {
        if ($arg_flashver != "my_secret_flashver") {
            rewrite ^.*$ fallback? permanent;
        }
        return 200;
    }

#### on_play
Syntax: `on_play url`  
Context: rtmp, server, application  

Sets HTTP play callback. Each time a clients issues play command
an HTTP request is issued asynchronously and command processing is
suspended until it returns result code. HTTP result code is then 
analyzed.

* HTTP 2xx code continues RTMP session
* HTTP 3xx redirects RTMP to another stream whose name is taken from 
`Location` HTTP response header. If new stream name is started with `rtmp://`
then remote relay is created instead. Relays require that IP address is
specified instead of domain name and only work with nginx versions
greater than 1.3.10. See also `notify_relay_redirect`.
* Otherwise RTMP connection is dropped

Redirect example

    http {
        ...
        location /local_redirect {
            rewrite ^.*$ newname? permanent;
        }
        location /remote_redirect {
            # no domain name here, only ip
            rewrite ^.*$ rtmp://192.168.1.123/someapp/somename? permanent;
        }
        ...
    }

    rtmp {
        ...
        application myapp1 {
            live on;
            # stream will be redirected to 'newname'
            on_play http://localhost:8080/local_redirect;
        }
        application myapp2 {
            live on;
            # stream will be pulled from remote location
            # requires nginx >= 1.3.10
            on_play http://localhost:8080/remote_redirect;
        }
        ...
    }

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
callback on publish command. Instead of remote pull push is performed in
this case.

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

#### on_update
syntax: `on_update url`  
context: rtmp, server, application  
  
Set update callback. This callback is called with period of 
`notify_update_timeout`. If a request returns HTTP result other
than 2xx connection is terminated. This can be used to synchronize
expired sessions. Additional `time` argument is passed to this handler
which is the number of seconds since play/publish call.

    on_update http://example.com/update;

#### notify_update_timeout
syntax: `notify_update_timeout timeout`  
context: rtmp, server, application  

Sets timeout between `on_update` callbacks. Default is 30 seconds.

    notify_update_timeout 10s;
    on_update http://example.com/update;

#### notify_update_strict
syntax: `notify_update_strict on|off`  
context: rtmp, server, application  

Toggles strict mode for `on_update` callbacks. Default is off.
When turned on all connection errors, timeouts as well as HTTP parse
errors and empty responses are treated as update failures and lead
to connection termination. When off only valid HTTP response codes
other that 2xx lead to failure.

    notify_update_strict on;
    on_update http://example.com/update;

#### notify_relay_redirect
syntax: `notify_relay_redirect on|off`  
context: rtmp, server, application  

Enables local stream redirect for `on_play` and `on_publish` remote
redirects. New stream name is RTMP URL used for remote redirect.
Default is off.

    notify_relay_redirect on;

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

In `http{}` section set up the following location for clients to play HLS.

    http {
        ...
        server {
            ...
            location /hls {
                types {
                    application/vnd.apple.mpegurl m3u8;
                }
                alias /tmp/hls;
            }
        }
    }

#### hls_path
Syntax: `hls_path path`  
Context: rtmp, server, application  

Sets HLS playlist and fragment directory. This directory should
exist before NGINX starts.

#### hls_fragment
Syntax: `hls_fragment time`  
Context: rtmp, server, application  

Sets HLS fragment length. Defaults to 5 seconds.

#### hls_playlist_length
Syntax: `hls_playlist_length time`  
Context: rtmp, server, application  

Sets HLS playlist length. Defaults to 30 seconds.

    hls_playlist_length 10m;

#### hls_sync
Syntax: `hls_sync time`  
Context: rtmp, server, application  

Sets HLS timestamp synchronization threshold. Default is 2ms.
This feature prevents crackling noises after conversion
from low-resolution RTMP (1KHz) to high-resolution MPEG-TS (90KHz).

    hls_sync 100ms;

#### hls_continuous
Syntax: `hls_continuous on|off`  
Context: rtmp, server, application  

Toggles HLS continuous mode. In this mode HLS sequence number
is started from where it stopped last time. Old fragments are
keeped. Default is off.

    hls_continuous on;

#### hls_nested
Syntax: `hls_nested on|off`  
Context: rtmp, server, application  

Toggles HLS nested mode. In this mode a subdirectory
of `hls_path` is created for each stream. Playlist
and fragments are created in that subdirectory.
Default is off.

    hls_nested on;

#### hls_cleanup
Syntax: `hls_cleanup on|off`  
Context: rtmp, server, application  

Toggles HLS cleanup. By default the feature is on.
In this mode nginx cache manager process removes old
HLS fragments and playlists from HLS directory.

    hls_cleanup off;

## Access log

#### access_log
Syntax: `access_log off|path [format_name]`  
Context: rtmp, server, application  

Sets access log parameters. Logging is turned on by default.
To turn it off use `access_log off` directive. By default access logging
is done to the same file as HTTP access logger (`logs/access.log`).
You can specify another log file path in `access_log` directive.
Second argument is optional. It can be used to specify logging format by name.
See `log_format` directive for more details about formats.

    log_format new '$remote_addr';
    access_log logs/rtmp_access.log new;
    access_log logs/rtmp_access.log;
    access_log off;

#### log_format
Syntax: `log_format format_name format`  
Context: rtmp  

Creates named log format. Log formats look very much the same as nginx HTTP log
formats. Several variables are supported within log format:
* `connection` - connection number
* `remote_addr` - client address
* `app` - application name
* `name` - last stream name
* `args` - last stream play/publish arguments
* `flashver` - client flashVer
* `swfurl` - client swfUrl
* `tcurl` - client tcUrl
* `pageurl` - client pageUrl
* `command` - play/publish commands sent by client: `NONE`, `PLAY`, `PUBLISH`, `PLAY+PUBLISH`
* `bytes_sent` - number of bytes sent to client
* `bytes_received` - number of bytes received from client
* `time_local` - local time at the end of client connection
* `session_time` - connection duration in seconds
* `session_readable_time` - connection duration in human-readable format

Default log format has the name `combined`. Here's the definition of this format

    $remote_addr [$time_local] $command "$app" "$name" "$args" - 
    $bytes_received $bytes_sent "$pageurl" "$flashver" ($session_readable_time)

## Limits

#### max_connections
Syntax: `max_connections number`  
Context: rtmp, server, application  

Sets maximum number of connections for rtmp engine. Off by refault.

    max_connections 100;

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