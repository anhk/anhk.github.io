---
layout: post
title: 检查Docker容器的连接数命令
---
{% raw %}
```bash
$ nsenter -t $(docker inspect -f '{{ .State.Pid }}' container_name_or_id) -n netstat
```
{% endraw %}