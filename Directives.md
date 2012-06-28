### Core
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
Creates RTMP application. Unlike http locations application name
should not be specified as a pattern but 

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
    ping_timeout 30s; # wait 15 sec for ping reply

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

### Access

allow

deny

### Exec

exec

respawn

respawn_timeout

### Live

live

stream_buckets

### Record

record

record_path

record_suffix

record_unique

record_max_size

record_max_frames

record_interval

on_record_done

### Relay

push

pull

relay_buffer

### Statistics

rtmp_stat

rtmp_stat_stylesheet