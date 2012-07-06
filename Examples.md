### Simple Video-on-Demand

    rtmp {
        server {
            listen 1935;
            application vod {
                play /var/flvs;
            }
        }
    }

### Simple live broadcast service

    rtmp {
        server {
            listen 1935;
            application live {
                live on;
            }
        }
    }

### Re-translate remote stream

    rtmp {
        server {
            listen 1935;
            application tv {
                live on;
                pull rtmp://cdn.example.com:443/programs/main pageUrl=http://www.example.com/index.html name=maintv;
            }
        }
    }

### Stream your X screen through RTMP

    ffmpeg -f x11grab -follow_mouse centered -r 25 -s cif -i :0.0 -f flv rtmp://localhost/myapp/screen