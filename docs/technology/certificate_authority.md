# CA及签发证书

## 前言

>使用https需配置证书，一般CA机构获取的证书是收费的，出于测试的话可使用openssl或keytool制作证书。


## 使用 openssl 生成证书

### 私钥PEM转DER
```bash
openssl rsa -in server_rsa_private.key -pubout -out server.pem
```


### 生成 RSA 私钥和自签名证书
```bash
openssl req -utf8 -newkey rsa:2048 -nodes -keyout server_rsa_private.key -x509 -days 3650 -out server.crt -subj "/C=CN/ST=湖北/L=武汉/O=武汉XXXX有限公司/OU=IT/CN=localhost/PostalCode=430000"
```


##  Let's Encrypt生成免费证书

### linux
[acme](https://github.com/acmesh-official/acme.sh)

### window
[acme-win](https://github.com/win-acme/win-acme)

### golang实现版本
[go-acme](https://github.com/go-acme/lego.git)

