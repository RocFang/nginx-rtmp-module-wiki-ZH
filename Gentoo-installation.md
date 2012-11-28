
## Download module source code
You have many options:
* Get the zip at https://github.com/arut/nginx-rtmp-module/archive/master.zip
* Or much better, do a git clone (see options in top of https://github.com/arut/nginx-rtmp-module)

## Emerge nginx with nginx-rtmp-module
`NGINX_ADD_MODULES="/path/to/nginx-rtmp-module" emerge -uva nginx`

To make this change permanent see:
http://wiki.gentoo.org/wiki/Knowledge_Base:Overriding_environment_variables_per_package
