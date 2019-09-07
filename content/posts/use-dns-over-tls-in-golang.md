---
title: 使 Go App 使用 DNS over TLS
date: '2018-12-06T16:45:00+08:00'
categories:
- Golang
---
最近使用 Golang 写了一个应用, 但是在使用的时候发现有些域名在中国被污染了. 于是使用 `DoT (DNS over TLS)` 解决了这个问题


<!--more-->


## 具体实现
```go
func newResolver(serverName string, addr string) *net.Resolver {
	var dialer net.Dialer
	tlsConfig := &tls.Config{
		ServerName: serverName,
		ClientSessionCache: tls.NewLRUClientSessionCache(32),

		InsecureSkipVerify: false,
	}

	return &net.Resolver{
		PreferGo: true,
		Dial: func (context context.Context, _, address string) (net.Conn, error) {
			conn, err := dialer.DialContext(context, "tcp", addr)
			if err != nil {
				return nil, err
			}

			_ = conn.(*net.TCPConn).SetKeepAlive(true)
			_ = conn.(*net.TCPConn).SetKeepAlivePeriod(10 * time.Minute)
			return tls.Client(conn, tlsConfig), nil
		},
	}

}

func init() {
	net.DefaultResolver = newResolver('cloudflare-dns.com', '1.0.0.1:853')
}
```

## 原理解析
Golang 在解析域名的时候会通过 `net.DefaultResolver` 来进行解析, 只需要写一个支持的 `net.Resolver` 就可以做到 DoT 了, 并且 Golang 自带了 tls 库可以方便的完成这一操作
