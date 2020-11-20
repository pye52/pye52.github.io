---
title: 转换bks证书
date: 2017-08-31 12:10:20
tags:
- cert
---

## crt转换bks证书

下载[bcprov](http://www.bouncycastle.org/latest_releases.html)，放至java/jre/lib/ext。

执行以下命令：(D:\server.crt为证书地址，D:\server.bks为证书生成地址)

keytool -import -alias serverkey -file D:\server.crt -keystore D:\server.bks -storetype BKS -provider org.bouncycastle.jce.provider.BouncyCastleProvider

## pem转换bks证书

命令行执行：(D:\server.pem为证书地址，D:\server.bks为证书生成地址)

keytool -importcert -v -trustcacerts -file D:\server.pem -alias ca -keystore D:\server.bks -storetype BKS -provider org.bouncycastle.jce.provider.BouncyCastleProvider  