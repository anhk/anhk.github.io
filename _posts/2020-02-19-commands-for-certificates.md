---
layout: post
title: 生成OpenSSL证书的常用命令
category: SSL数字证书
---

这里汇集了生成OpenSSL证书的常用命令
<!--more-->

## RSA证书

### 1. 快速生成自签名证书

```bash
# 快速生成带密码的一对证书
openssl req -new -x509 -keyout ca.key -out ca.crt -subj "/C=CN/ST=Beijing/L=Beijing/O=anhk.inc./OU=anhk.cc/CN=*.anhk.cc" -days 3650

# 快速生成不带密码的一对证书
openssl req -nodes -new -x509 -keyout ca.key -out ca.crt -subj "/C=CN/ST=Beijing/L=Beijing/O=anhk.inc./OU=anhk.cc/CN=*.anhk.cc" -days 3650

```



### 2. 分步骤生成自签名证书

```bash
# 生成不带密码的私钥
openssl genrsa -out ca.key 2048

# 生成带密码的私钥，与上一行命令二选一
openssl genrsa -aes256 -out ca.key 2048

# 生成证书请求文件
openssl req -new -key ca.key -out ca.csr -subj "/C=CN/ST=Beijing/L=Beijing/O=anhk.inc./OU=anhk.cc/CN=*.anhk.cc"

# 自签名证书
openssl x509 -req -days 3650 -sha1 -extensions v3_ca -signkey ca.key -in ca.csr -out ca.crt
```



### 3. 使用CA证书签名服务器证书

```bash
# 生成服务器证书的私钥
openssl genrsa -out server.key 2048 【-pass:123456】

# 生成证书请求文件
openssl req -new -key server.key -out server.csr -subj "/C=CN/ST=Beijing/L=Beijing/O=anhk.inc./OU=anhk.cc/CN=*.anhk.cc"

# 使用CA证书签名
openssl x509 -req -days 3650 -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt
```



### 4. 使用CA证书签发多域名证书 SANS

```bash
# 快速生成私钥及CSR文件
openssl req -newkey rsa:2048 -nodes -keyout server.key -subj "/C=CN/ST=BJ/L=BJ/O=anhk.inc./CN=*.anhk.cc" -out server.csr

# 使用CA证书签名
openssl x509 -req -extfile <(printf "subjectAltName=DNS:anhk.cc,DNS:www.anhk.cc") -days 3650 -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt
```



### 5. 证书内容查看

```bash
openssl rsa -in ca.key -noout -text
openssl req -in ca.csr -noout -text
openssl x509 -in ca.crt -noout -text
```


### 6. 私钥脱密

```bash
openssl rsa -in ca.key -out ca2.key
```

###  7. 快速生成自签名证书
```bash
# DV
openssl genrsa -out my.dv.key 2048
openssl req -new -key my.dv.key -out my.dv.csr -subj "/CN=dv.anhk.cc"
openssl x509 -req -days 3650 -sha1 -signkey my.dv.key -in my.dv.csr -out my.dv.crt

# OV
openssl genrsa -out my.ov.key 2048
openssl req -new -key my.ov.key -out my.ov.csr -subj "/C=CN/ST=Beijing/L=Beijing/O=anhk.inc./OU=anhk.cc/CN=ov.anhk.cc"
openssl x509 -req -days 3650 -sha1 -signkey my.ov.key -in my.ov.csr -out my.ov.crt

# EV
openssl genrsa -out my.ev.key 2048
openssl req -new -key my.ev.key -out my.ev.csr -subj "/businessCategory=Private/serialNumber=5157550/jurisdictionC=US/CN=anhk.inc./O=anhk.inc./C=FR"
openssl x509 -req -days 3650 -sha1 -signkey my.ev.key -in my.ev.csr -out my.ev.crt
```

###  8. 证书格式转换

```bash
# PEM to PFX/P12
openssl pkcs12 -export -inkey my.dv.key -in my.dv.crt  -out my.dv.pfx

# PFX/P12 to PEM
openssl pkcs12 -in my.dv.pfx -nodes -out my.dv.pem
openssl rsa -in my.dv.pem -out my.dv.key
openssl x509 -in my.dv.pem -out my.dv.crt

# PKCS#1 to PKCS#8
openssl pkcs8 -topk8 -nocrypt -in p1.key -out p8.key

# PKCS#8 to PKCS#1
openssl rsa -in p8.key -out p1.key
```



## ECC证书
### 1. 查看支持算法
```bash
openssl ecparam -list_curves
```

### 2. 生成私钥
```bash
openssl ecparam -name secp521r1 -genkey -noout -param_enc explicit -out my.key.pem
```

### 3. 生成CSR
```bash
openssl req -new -sha256 -key my.key.pem -out my.csr.pem \
	-subj "/C=CN/ST=Beijing/L=Beijing/O=anhk.inc./OU=anhk.cc/CN=*.anhk.cc"
```

### 4. 签发（自签名）
```bash
openssl x509 -req -sha256 -days 365 -in my.csr.pem -signkey my.key.pem -out my.crt.pem
```

### 5. 证书内容查看
```bash
openssl ecparam -in my.key.pem -text -noout
openssl req -in my.csr.pem -noout -text
openssl x509 -in my.crt.pem -text -noout
```

### 6. 快速自签名证书
```bash
openssl ecparam -name prime256v1 -genkey -param_enc named_curve -out my.key.pem
openssl req -new -x509 -key my.key.pem -out my.crt.pem -days 730 \
-subj "/C=CN/ST=Beijing/L=Beijing/O=anhk.inc./OU=anhk.cc/CN=*.anhk.cc"
```

### 7. 将EC私钥压缩
```bash
openssl ec -in my.key.pem  -conv_form compressed -out my.key.compressed.pem
```

### 8. 将EC私钥转换为PKCS#8格式
```bash
openssl pkcs8 -nocrypt -in 3.key -topk8 -out 4.key
```

### EC私钥操作
```bash
# 从EC私钥里获取Parameters
openssl asn1parse -i -in priv.key
openssl ecparam -in priv.key -name prime256v1
# 从EC私钥里获取原始私钥
openssl ec -in priv.key
```





##  证书用法限制

```bash
openssl x509 -req -days 3650 -sha256 -extensions v3_ca -extfile <(printf "[v3_ca]\n     \
        basicConstraints = CA:TRUE\nkeyUsage = digitalSignature, nonRepudiation,        \
        keyEncipherment, dataEncipherment, keyAgreement, keyCertSign, cRLSign,          \
        encipherOnly, decipherOnly \n                                                   \
        extendedKeyUsage=critical,serverAuth,clientAuth,codeSigning,emailProtection,    \
        timeStamping,msCodeInd,msCodeCom,msCTLSign,msSGC,msEFS,nsSGC\n                  \
        subjectAltName=DNS:anhk.cc,DNS:www.anhk.cc")                                    \
        -signkey ca.key -in ca.csr -out ca.crt
```