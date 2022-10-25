---
layout: post
title: Golang使用pprof优化性能
category: [Golang]
---

使用pprof包来为golang程序检查性能损耗点

<!--more-->

# 进程启动/退出时输出pprof文件

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

# 使用HTTP包输出pprof文件

```go
import _ "net/http/pprof"

go http.ListenAndServe("0.0.0.0:8000", nil)
```

针对自定义Mux提供手动添加路由：
```go
r.HandleFunc("/debug/pprof/", pprof.Index)
r.HandleFunc("/debug/pprof/cmdline", pprof.Cmdline)
r.HandleFunc("/debug/pprof/profile", pprof.Profile)
r.HandleFunc("/debug/pprof/symbol", pprof.Symbol)
r.HandleFunc("/debug/pprof/trace", pprof.Trace)
```

# 生成svg图

安装 `graphviz`
```bash
$ apt install -y graphviz
```

生成SVG图

```bash
$ go tool pprof [binary] pprof.file
$ svg
```