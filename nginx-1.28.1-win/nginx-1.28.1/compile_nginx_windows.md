# 编译Windows版Nginx 1.28.1包含more_set_headers和GeoIP2模块

## 环境要求

1. **Visual Studio**：推荐使用Visual Studio 2022或更高版本
2. **Perl**：ActivePerl或Strawberry Perl
3. **GNU环境**：MSYS2或Cygwin
4. **Git**：用于下载第三方模块源码
5. **CMake**：可选，用于编译某些依赖库

## 编译步骤

### 1. 安装必要的编译工具

- 安装Visual Studio，确保选择C++开发组件
- 安装Perl
- 安装MSYS2，并在MSYS2中安装必要的工具：
  ```bash
  pacman -S base-devel mingw-w64-x86_64-toolchain
  ```

### 2. 下载源码和依赖库

- **Nginx源码**：https://nginx.org/download/nginx-1.28.1.tar.gz
- **依赖库**：
  - OpenSSL：https://www.openssl.org/source/openssl-3.2.0.tar.gz
  - zlib：https://zlib.net/zlib-1.3.1.tar.gz
  - PCRE2：https://github.com/PCRE2Project/pcre2/releases/download/pcre2-10.42/pcre2-10.42.tar.gz
- **第三方模块**：
  - more_set_headers：`git clone https://github.com/openresty/headers-more-nginx-module.git`
  - GeoIP2：`git clone https://github.com/leev/ngx_http_geoip2_module.git`

### 3. 配置编译选项

在MSYS2环境中，进入Nginx源码目录，执行以下命令配置编译选项：

```bash
auto/configure --with-cc=cl --builddir=objs --prefix= --conf-path=conf/nginx.conf --pid-path=logs/nginx.pid --http-log-path=logs/access.log --error-log-path=logs/error.log --sbin-path=nginx.exe --http-client-body-temp-path=temp/client_body_temp --http-proxy-temp-path=temp/proxy_temp --http-fastcgi-temp-path=temp/fastcgi_temp --http-scgi-temp-path=temp/scgi_temp --http-uwsgi-temp-path=temp/uwsgi_temp --with-pcre=../pcre2-10.42 --with-zlib=../zlib-1.3.1 --with-openssl=../openssl-3.2.0 --with-openssl-opt=no-asm --with-select_module --with-poll_module --with-http_ssl_module --with-http_realip_module --with-http_addition_module --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_stub_status_module --with-http_auth_request_module --with-http_slice_module --with-mail --with-mail_ssl_module --with-stream --with-stream_ssl_module --with-stream_realip_module --with-stream_ssl_preread_module --with-threads --add-module=../headers-more-nginx-module --add-module=../ngx_http_geoip2_module
```

### 4. 编译Nginx

在Visual Studio命令提示符中，进入Nginx源码目录，执行以下命令编译：

```bash
nmake -f objs/Makefile
```

### 5. 测试编译结果

编译成功后，在`objs`目录中会生成`nginx.exe`文件。可以执行以下命令测试：

```bash
nginx.exe -V
```

该命令会显示Nginx的版本和编译选项，确认是否包含了所需的模块。

## 注意事项

1. 确保所有依赖库的版本与Nginx兼容
2. 编译过程中可能需要调整编译选项，根据实际情况进行修改
3. 如果遇到编译错误，可以查看`objs/autoconf.err`文件获取详细信息
4. 编译完成后，需要将必要的配置文件和依赖库复制到合适的位置

## 替代方案

如果编译过程中遇到困难，可以考虑以下替代方案：

1. 使用预编译的Nginx二进制文件，并通过动态模块的方式添加所需模块
2. 使用Docker容器，在Linux环境中编译Windows版Nginx
3. 使用第三方提供的包含所需模块的预编译Nginx二进制文件

## 动态模块编译

如果使用预编译的Nginx二进制文件，可以通过以下步骤编译动态模块：

1. 确保下载的Nginx二进制文件与源码版本完全一致
2. 在MSYS2环境中，配置编译选项时添加`--with-compat`和`--add-dynamic-module`选项
3. 编译生成`.so`文件（Windows上为`.dll`文件）
4. 将生成的模块文件复制到Nginx的`modules`目录中
5. 在Nginx配置文件中添加`load_module`指令加载模块

```nginx
load_module modules/ngx_http_headers_more_filter_module.so;
load_module modules/ngx_http_geoip2_module.so;
```