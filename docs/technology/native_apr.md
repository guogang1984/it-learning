# tomcat native警告问题（Mac OS）
>问题描述：Mac OS下 Tomcat启动老是会出现如下警告:

>The APR based Apache Tomcat Native library which allows optimal performance in production environments was not found on the java.library.path:...

>总结一下解决方法。

## 源码下载并编译OpenSSL
>编译并安装到 /usr/local/opt/ 目录

```bash
# 拉取源代码
git clone https://github.com/openssl/openssl.git
# 或者国内镜像
git clone https://gitee.com/mirrors/openssl.git

# 切换指定版本，比如1.1.1g
git checkout OpenSSL_1_1_1g

# mac下编译
./Configure darwin64-x86_64-cc  --prefix=/usr/local/opt/openssl@1.1.1g --openssldir=/usr/local/ssl
sudo make && sudo make install

```

## 下载并编译APR, APR-util
>下载APR,APR-util [https://apr.apache.org/download.cgi](https://apr.apache.org/download.cgi)

```bash
cd /<your_apr_dir>
CFLAGS='-arch x86_64' ./configure
sudo make && make install
```

```bash
cd /<your_apr-util_dir>
CFLAGS='-arch x86_64' ./configure --with-apr=/usr/local/apr/bin/apr-1-config
sudo make && make install
```

## 下载编译Tomcat
>下载 [http://tomcat.apache.org/download-native.cgi](http://tomcat.apache.org/download-native.cgi)

```bash
cd /<your_tomcat-native_dir>/native
CFLAGS='-arch x86_64' ./configure --with-apr=/usr/local/apr --with-ssl=/usr/local/opt/openssl --with-java-home=/Library/Java/JavaVirtualMachines/jdk1.8.0_202.jdk/Contents/Home/
sudo make && make install
```




