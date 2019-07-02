## 安装依赖

### pcre（对正则表达式的支持）

```
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.38.tar.gz
tar -zxvf pcre-8.38.tar.gz
cd pcre-8.38
./configure --prefix=/usr/local/pcre
sudo make && sudo make install
```

### zlib（对 gzip 压缩的支持）

```
wget wget http://www.zlib.net/zlib-1.2.8.tar.gz
tar -zxvf zlib-1.2.8.tar.gz
cd zlib-1.2.8
./configure --prefix=/usr/local/zlib
sudo make && sudo make install
```

### openssl（对 ssl 协议的支持）

```
wget ftp://ftp.openssl.org/source/openssl-1.0.2g.tar.gz
tar -zxvf openssl-1.0.2g.tar.gz
cd openssl-1.0.2g
./config --prefix=/usr/local/openssl
sudo make && sudo make install
```

### nginx

```
wget http://nginx.org/download/nginx-1.9.9.tar.gz
tar -zxvf nginx-1.9.9.tar.gz
cd nginx-1.9.9

./configure \
--prefix=/usr/local/nginx \
--sbin-path=/usr/sbin/nginx \
--conf-path=/usr/local/nginx/nginx.conf \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--pid-path=/var/run/nginx/nginx.pid  \
--lock-path=/var/lock/nginx.lock \
--with-openssl=<openssl_dir> \
--with-pcre=<pcre_dir> \
--with-zlib=<zlib_dir>

sudo make && sudo make install
```

## 运行

```
sudo /usr/sbin/nginx -c  /usr/local/nginx/nginx.conf
打开 http://localhost，看见welcome代表成功。
```
