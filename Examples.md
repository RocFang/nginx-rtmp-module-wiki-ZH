## Simple Video-on-Demand

    rtmp {
        server {
            listen 1935;
            application vod {
                play /var/flvs;
            }
        }
    }

## FFmpeg streaming your X screen through RTMP

    ffmpeg -f x11grab -follow_mouse centered -r 25 -s cif -i :0.0 -f flv rtmp://localhost/myapp/screen