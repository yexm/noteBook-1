# CentOs 7编译安装Freeswitch

```bash
yum install -y http://files.freeswitch.org/freeswitch-release-1-6.noarch.rpm epel-release
yum install -y git alsa-lib-devel autoconf automake bison broadvoice-devel bzip2 curl-devel db-devel e2fsprogs-devel flite-devel g722_1-devel gcc-c++ gdbm-devel gnutls-devel ilbc2-devel ldns-devel libcodec2-devel libcurl-devel libedit-devel libidn-devel libjpeg-devel libmemcached-devel libogg-devel libsilk-devel libsndfile-devel libtheora-devel libtiff-devel libtool libuuid-devel libvorbis-devel libxml2-devel lua-devel lzo-devel mongo-c-driver-devel ncurses-devel net-snmp-devel openssl-devel opus-devel pcre-devel perl perl-ExtUtils-Embed pkgconfig portaudio-devel postgresql-devel python26-devel python-devel soundtouch-devel speex-devel sqlite-devel unbound-devel unixODBC-devel wget which yasm zlib-devel erlang lksctp-tools-devel lksctp* libsctp*
 ```

切换至源码目录，下载1.6版本源码

```bash
cd /usr/local/src
git clone -b v1.6 https://freeswitch.org/stash/scm/fs/freeswitch.git freeswitch
```

编译Freeswitch

```bash
cd /usr/local/src/freeswitch
./bootstrap.sh -j
./configure -C --enable-portable-binary --enable-sctp\
            --prefix=/usr --localstatedir=/var --sysconfdir=/etc \
            --with-gnu-ld --with-python --with-erlang --with-openssl \
            --enable-core-odbc-support --enable-zrtp \
            --enable-core-pgsql-support \
            --enable-static-v8 --disable-parallel-build-v8
make -j
make -j install
make -j cd-sounds-install
make -j cd-moh-install
```

## 其他支持组件

### 编译x264 最新版本依赖nasm 2.13需要编译安装

```bash
./configure --enable-static --enable-pic
make && make install
```

### 编译rtmpdump

```bash
make && make install
```

### 编译ffmpeg

```bash
cd ffmpeg-3.4.4
export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH
./configure --arch=x86_64 --target-os=linux --enable-cross-compile --enable-libx264 --enable-gpl --enable-avresample  --extra-libs="-ldl" --enable-shared
make
make install
```

### 配置Verto

使用verto需要https的支持

生成ssl证书
wget http://files.freeswitch.org/downloads/ssl.ca-0.1.tar.gz
解压 ssl.ca-0.1.tar.gz

```bash
cd ssl.ca-0.1

perl -i -pe 's/md5/sha256/g' *.sh
perl -i -pe 's/1024/4096/g' *.sh
./new-root-ca.sh
./new-server-cert.sh verto
./sign-server-cert.sh verto
./new-user-cert.sh 
cat verto.crt verto.key > wss.pem
cat wss.pem 
mkdir /usr/local/freeswitch/certs
mv verto.crt wss.pem verto.key /usr/local/freeswitch/certs/
mv verto.crt self.verto.crt
mv self.verto.crt wss.crt
mv verto.key wss.key
```

安装nginx

```bash
yum install nginx
systemctl enable nginx
systemctl start nginx
systemctl status nginx
vim /etc/nginx/nginx.conf
配置nginx.conf
     server {
         listen 443;
         server_name chenzy02.sss.com;
         ssl on;
         ssl_certificate /usr/local/freeswitch/certs/wss.crt;
         ssl_certificate_key /usr/local/freeswitch/certs/wss.key;
         ssl_session_timeout 5m;
         ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
         ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
         ssl_prefer_server_ciphers on;

     location / {
         root /usr/share/nginx/html; #站点目录
         index index.html index.htm;
         }
     }
nginx -t
nginx -s reload
systemctl restart nginx
```

编译前端：

```bash
cd /use/local/src/freeswitch/html5/verto/verto_communicator
```

安装npm yum install -y npm
使用npm安装bower和grunt

```bash
npm install -g browser
npm install -g grunt

npm install
bower install -u root
grunt
```

拷贝dist目录下所有文件至nginx指定的root path下。
重启nginx，尝试访问。