---
layout: post
title: tcpdump侦听多个端口
---
```bash
$ tcpdump -i any tcp and port 80 or port 443
# or 
$ tcpdump -i any tcp and port '(80 or 443)'
```
