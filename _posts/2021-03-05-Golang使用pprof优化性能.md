---
layout: post
title: Golang使用pprof优化性能
category: [Golang]
---

使用pprof包来为golang程序检查性能损耗点

<!--more-->


```golang

func StartProfile() {
	{ // 信号处理
		sigs := make(chan os.Signal, 1)
		signal.Notify(sigs, syscall.SIGINT, syscall.SIGTERM)
		go func() {
			<-sigs
			pprof.StopCPUProfile()
			os.Exit(0)
		}()
	}
	{ // 启动pprof
		pwd, err := os.Getwd()
		if err != nil {
			panic(err)
		}
		fileName := filepath.Join(pwd, "pprof.file")
		f, err := os.Create(fileName)
		if err != nil {
			panic(err)
		}

		if err := pprof.StartCPUProfile(f); err != nil {
			panic(err)
		}
	}
}
```

安装 `graphviz`
```bash
$ apt install -y graphviz
```

生成SVG图

```bash
$ go tool pprof [binary] pprof.file
$ svg
```