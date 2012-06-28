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

application
syntax: application name { ... }  
context: server  
Creates RTMP application

    server {
        listen 1935;
        application myapp {
        }
    }


so_keepalive

timeout

ping

ping_timeout

max_streams

ack_window

chunk_size

max_queue

max_message

out_queue

out_cork

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