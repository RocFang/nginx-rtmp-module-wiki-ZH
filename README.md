# 基于 Nginx 的流媒体服务器

## 说明

1. nginx-rtmp是一个非常优秀的流媒体系统，arut精湛的编程技艺使得无论是用该项目来学习
流媒体服务器设计，还是基于其做二次开发，都非常合适。
2. 由于arut精力早就不在本项目上，所以很遗憾，这个框架有各种这样那样的问题。
这些问题主要来自这几个方面：（1）大量bug没有修复，当然对于一个服务器软件来说，
这是很正常的。（2）未完全完成的部分。例如对多进程的支持，其实还有很多工作没有完成，
导致原框架的多进程在实际中几乎不可用。（3）设计的缺陷。nginx-rtmp中也有一些在设计
之初，arut就没有考虑到或考虑欠周的情况。所以，一方面国内很多公司基于其做流媒体系统，
但另一方面，应该没有谁会直接用它来提供服务，都得花大力气去深度完善甚至改造。
3. 本wiki是nginx-rtmp官方wiki的一个fork，仅是对原框架的说明，会有一些新加的文档。
但不会涉及到bug的修复和新的功能。

## nginx-rtmp 模块

### 项目博客

arut一共维护了两份开发日志：
* http://nginx-rtmp.blogspot.com
* https://rarut.wordpress.com

arut起先在wordpress.com上面做开发记录，后来迁移到了blogspot.com上，所以，如果不是对
评论特别感兴趣的话，阅读第一个地址就够了。

这里必须要强调一下，arut的这些开发日志有很高的价值，通过这些文章，我们可以了解很多代码
背后的东西。强烈建议精读几遍。

### 配置指令说明

  * 英文: https://github.com/arut/nginx-rtmp-module/wiki/Directives
  * 中文: [配置指令集](/Directives.md)

### Google 讨论组

  * 英文: https://groups.google.com/group/nginx-rtmp
  * 俄文: https://groups.google.com/group/nginx-rtmp-ru

### 特性

* 支持RTMP/HLS/MPEG-DASH直播

* 支持基于RTMP协议的FLV/MP4点播,既支持本地文件点播，也支持通过http从远程拉取文件做点播

* 提供基于relay的分布式支持，主要有两种模式：push和pull

* 支持将直播流录制为flv文件落地到磁盘

* 支持的编码格式是H264/AAC

* 支持使用FFmepg做在线转码

* 提供了HTTP回调支持(publish/play/record/update等)

* 支持通过事件触发来运行外部程序(exec模块)

* 支持通过HTTP请求来控制录制、断开客户端的动作

* 通过精心设计的buffer,使得本系统有着极高的运行效率和极小的内存占用

* 经过测试的客户端包括Wirecast, FMS, Wowza,JWPlayer, FlowPlayer, StrobeMediaPlayback,
  ffmpeg, avconv, rtmpdump, flvstreamer等等

* 支持通过http请求获取系统内实时的统计数据，统计数据用xml/xsl的形式展现出来

* 兼容Linux/FreeBSD/MacOS/Windows

注意：上面提到的控制和统计功能，分别是用两个http模块来实现的，在多进程的时候这又是
一个雷区:一个http请求只能落在一个worker进程上面。所以，再次强调，如果不进行二次开发，
请不要在多进程时使用这两个功能。

### 编译

跟所有的nginx第三方模块一样，nginx-rtmp通过如下的方式进行编译:在nginx源码目录下执行

    ./configure --add-module=/path/to/nginx-rtmp-module
    make
    make install

为了支持debug模式，在configure的时候带上 `--with-debug`参数

    ./configure --add-module=/path/to-nginx/rtmp-module --with-debug

[更多关于debug日志的内容](https://github.com/arut/nginx-rtmp-module/wiki/Debug-log)

### RTMP url格式

    rtmp://rtmp.example.com/app[/name]

app -  匹配配置文件中的某个application{}块

name - 由各程序自行处理和解释，可以为空。一般用它来做流名。


### 多进程支持

nginx-rtmp通过auto_push功能来实现对多进程的支持。大概方法是，每一个被publish到的worker
进程，通过unix域套接字将其接收到的流主动publish到其它各个worker进程。这个功能通过
rtmp_auto_push指令来开启。详见ngx_rtmp_auto_push_module模块。

注意：已经多次强调，nginx-rtmp的多进程支持并不完全，除非你已经非常熟悉源码并能做相应
的修改，否则永远不要在生产环境中开启多进程支持。

### 配置文件示例-单进程版

    rtmp {

        server {

            listen 1935;

            chunk_size 4000;

            # TV mode: one publisher, many subscribers
            application mytv {

                # enable live streaming
                live on;

                # record first 1K of stream
                record all;
                record_path /tmp/av;
                record_max_size 1K;

                # append current timestamp to each flv
                record_unique on;

                # publish only from localhost
                allow publish 127.0.0.1;
                deny publish all;

                #allow play all;
            }

            # Transcoding (ffmpeg needed)
            application big {
                live on;

                # On every pusblished stream run this command (ffmpeg)
                # with substitutions: $app/${app}, $name/${name} for application & stream name.
                #
                # This ffmpeg call receives stream from this application &
                # reduces the resolution down to 32x32. The stream is the published to
                # 'small' application (see below) under the same name.
                #
                # ffmpeg can do anything with the stream like video/audio
                # transcoding, resizing, altering container/codec params etc
                #
                # Multiple exec lines can be specified.

                exec ffmpeg -re -i rtmp://localhost:1935/$app/$name -vcodec flv -acodec copy -s 32x32
                            -f flv rtmp://localhost:1935/small/${name};
            }

            application small {
                live on;
                # Video with reduced resolution comes here from ffmpeg
            }

            application webcam {
                live on;

                # Stream from local webcam
                exec_static ffmpeg -f video4linux2 -i /dev/video0 -c:v libx264 -an
                                   -f flv rtmp://localhost:1935/webcam/mystream;
            }

            application mypush {
                live on;

                # Every stream published here
                # is automatically pushed to
                # these two machines
                push rtmp1.example.com;
                push rtmp2.example.com:1934;
            }

            application mypull {
                live on;

                # Pull all streams from remote machine
                # and play locally
                pull rtmp://rtmp3.example.com pageUrl=www.example.com/index.html;
            }

            application mystaticpull {
                live on;

                # Static pull is started at nginx start
                pull rtmp://rtmp4.example.com pageUrl=www.example.com/index.html name=mystream static;
            }

            # video on demand
            application vod {
                play /var/flvs;
            }

            application vod2 {
                play /var/mp4s;
            }

            # Many publishers, many subscribers
            # no checks, no recording
            application videochat {

                live on;

                # The following notifications receive all
                # the session variables as well as
                # particular call arguments in HTTP POST
                # request

                # Make HTTP request & use HTTP retcode
                # to decide whether to allow publishing
                # from this connection or not
                on_publish http://localhost:8080/publish;

                # Same with playing
                on_play http://localhost:8080/play;

                # Publish/play end (repeats on disconnect)
                on_done http://localhost:8080/done;

                # All above mentioned notifications receive
                # standard connect() arguments as well as
                # play/publish ones. If any arguments are sent
                # with GET-style syntax to play & publish
                # these are also included.
                # Example URL:
                #   rtmp://localhost/myapp/mystream?a=b&c=d

                # record 10 video keyframes (no audio) every 2 minutes
                record keyframes;
                record_path /tmp/vc;
                record_max_frames 10;
                record_interval 2m;

                # Async notify about an flv recorded
                on_record_done http://localhost:8080/record_done;

            }


            # HLS

            # For HLS to work please create a directory in tmpfs (/tmp/hls here)
            # for the fragments. The directory contents is served via HTTP (see
            # http{} section in config)
            #
            # Incoming stream must be in H264/AAC. For iPhones use baseline H264
            # profile (see ffmpeg example).
            # This example creates RTMP stream from movie ready for HLS:
            #
            # ffmpeg -loglevel verbose -re -i movie.avi  -vcodec libx264
            #    -vprofile baseline -acodec libmp3lame -ar 44100 -ac 1
            #    -f flv rtmp://localhost:1935/hls/movie
            #
            # If you need to transcode live stream use 'exec' feature.
            #
            application hls {
                live on;
                hls on;
                hls_path /tmp/hls;
            }

            # MPEG-DASH is similar to HLS

            application dash {
                live on;
                dash on;
                dash_path /tmp/dash;
            }
        }
    }

    # HTTP can be used for accessing RTMP stats
    http {

        server {

            listen      8080;

            # This URL provides RTMP statistics in XML
            location /stat {
                rtmp_stat all;

                # Use this stylesheet to view XML as web page
                # in browser
                rtmp_stat_stylesheet stat.xsl;
            }

            location /stat.xsl {
                # XML stylesheet to view RTMP stats.
                # Copy stat.xsl wherever you want
                # and put the full directory path here
                root /path/to/stat.xsl/;
            }

            location /hls {
                # Serve HLS fragments
                types {
                    application/vnd.apple.mpegurl m3u8;
                    video/mp2t ts;
                }
                root /tmp;
                add_header Cache-Control no-cache;
            }

            location /dash {
                # Serve DASH fragments
                root /tmp;
                add_header Cache-Control no-cache;
            }
        }
    }


### 配置文件示例-多进程版

    rtmp_auto_push on;
    worker_processes 4;

    rtmp {
        server {
            listen 1935;

            application mytv {
                live on;
            }
        }
    }
