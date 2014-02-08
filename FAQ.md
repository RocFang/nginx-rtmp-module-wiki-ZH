####RTMP stream is not played normally in IE, stream stops after several seconds.

Add this directive to fix the problem

    wait_video on;

####I use `pull` directive to get stream from remote location. That works for RTMP clients but does not work for HLS.

Currently HLS clients do not trigger any events. You cannot pull or exec when HLS client connects to server. However you can use static directives `exec_static`, `pull ... static` to pull the stream always. 