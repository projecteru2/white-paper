# 部署

## 线上部署

在 citadel 中进行线上部署，只需要遵循 citadel 文档即可。

## 本地部署

* 下载 [openresty-1.11.2.2](https://openresty.org/download/openresty-$OVERSION.tar.gz) 以及 [ngx_http_dyups_modules](https://github.com/yzprofile/ngx_http_dyups_module) 的源码;

* 编译选项: `./configure --with-http_realip_module --add-module=../ngx_http_dyups_module`

* 运行: `nginx -p /path/to/server -c /path/to/ELB/conf/release.conf`

* 安装可以参考 [alpine-openresty](http://gitlab.ricebook.net/footstone/alpine-openresty) , 运行可以参考 ELB 中的 `start.sh`。