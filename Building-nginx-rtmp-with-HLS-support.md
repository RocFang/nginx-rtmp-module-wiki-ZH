If ffmpeg on your system is too old or you have avconv instead ffmpeg follow
this guide to build nginx-rtmp with HLS support.

Build & install latest ffmpeg into /opt/ffmpeg

    cd /usr/build
    git clone git://source.ffmpeg.org/ffmpeg.git ffmpeg
    cd ffmpeg
    ./configure --prefix=/opt/ffmpeg --disable-yasm --enable-shared
    make
    make install

Download nginx-rtmp-module

    cd /usr/build
    git clone git://github.com/arut/nginx-rtmp-module.git

Download & build nginx

    wget http://nginx.org/download/nginx-1.2.4.tar.gz
    tar xzf nginx-1.2.4.tar.gz
    cd nginx-1.2.4
    ./configure --add-module=/usr/build/nginx-rtmp-module --add-module=/usr/build/nginx-rtmp-module/hls --with-cc-opt=-I/opt/ffmpeg/include --with-ld-opt='-L/opt/ffmpeg/lib -Wl,-rpath=/opt/ffmpeg/lib'
    make
    make install