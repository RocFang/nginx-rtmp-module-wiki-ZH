## Download module source code
You have many options:
* Get the zip at https://github.com/arut/nginx-rtmp-module/archive/master.zip
* Or much better, do a git clone (see options in top of https://github.com/arut/nginx-rtmp-module)

## Emerge nginx with nginx-rtmp-module
> NGINX_ADD_MODULES="/path/to/nginx-rtmp-module" emerge -va nginx

Replace `/path/to/` with the actual module's source path.
You can add more than one module that is not controlled by USE flags.

To make this change permanent see:
http://wiki.gentoo.org/wiki/Knowledge_Base:Overriding_environment_variables_per_package

## Configure nginx
Don't forget to include a rtmp section inside your nginx configuration file located at `/etc/nginx/nginx.conf`.

See:
* [Getting started](https://github.com/arut/nginx-rtmp-module/wiki/Getting-started-with-nginx-rtmp) We already have done _Download, build and install_ Gentoo style ;-)
* [More Examples](https://github.com/arut/nginx-rtmp-module/wiki/Examples)
* [Reference of all directives](https://github.com/arut/nginx-rtmp-module/wiki/Directives)
