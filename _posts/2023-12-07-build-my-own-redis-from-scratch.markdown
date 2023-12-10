---
layout: post
title:  "从零构建一个Redis"
date:   2023-12-07 11:15:00 +0800
categories: Jiang Jingwei
---

## 1.Hello Server/Client

Redis是一个server/client架构的系统，多个clients连接到一个server，server通过TCP连接接收请求并发送响应。

下面是一些system call的介绍：

1. `socket()`: 创建一个socket并返回对应的文件描述符`fd`，文件描述符是一个integer，指向Linux内核中的资源，比如TCP连接，磁盘上的文件，正在监听的端口等。

2. `bind()`: 将一个地址与socket关联起来。

3. `listen()`: 使socket能接受连接。

4. `accept()`: 当一个client尝试连接到正在监听的地址，`accept()`会返回一个`fd`，代表connection socket。

5. `read()`, `write()`: 从TCP connection中读取或向TCP connection中写入数据。

下面的代码是一个基本的server/client架构的系统的实现。server和client会分别向对方发送一个"hello"，并打印出来自对方的数据。

一些需要注意的细节：我们给socket绑定地址和端口时，调用了`ntohs()`和`ntohl()`，这是因为TCP协议规定数据应该使用大端序（big-endian）来传输。`ntohs()`和`ntohl()`的作用是将网络字节序（big-endian）转换为主机字节序（在英特尔平台上是little-endian）。

```cpp
// server.cpp

#include <stdint.h>
#include <stdlib.h>
#include <string.h>
#include <stdio.h>
#include <errno.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <netinet/ip.h>


static void msg(const char *msg) {
    fprintf(stderr, "%s\n", msg);
}

static void die(const char *msg) {
    int err = errno;
    fprintf(stderr, "[%d] %s\n", err, msg);
    abort();
}

static void do_something(int connfd) {
    char rbuf[64] = {};
    ssize_t n = read(connfd, rbuf, sizeof(rbuf) - 1);
    if (n < 0) {
        msg("read() error");
        return;
    }
    printf("client says: %s\n", rbuf);

    char wbuf[] = "world";
    write(connfd, wbuf, strlen(wbuf));
}

int main() {
    int fd = socket(AF_INET, SOCK_STREAM, 0);
    if (fd < 0) {
        die("socket()");
    }

    // this is needed for most server applications
    int val = 1;
    setsockopt(fd, SOL_SOCKET, SO_REUSEADDR, &val, sizeof(val));

    // bind
    struct sockaddr_in addr = {};
    addr.sin_family = AF_INET;
    addr.sin_port = ntohs(1234);
    addr.sin_addr.s_addr = ntohl(0);    // wildcard address 0.0.0.0
    int rv = bind(fd, (const sockaddr *)&addr, sizeof(addr));
    if (rv) {
        die("bind()");
    }

    // listen
    rv = listen(fd, SOMAXCONN);
    if (rv) {
        die("listen()");
    }

    while (true) {
        // accept
        struct sockaddr_in client_addr = {};
        socklen_t socklen = sizeof(client_addr);
        int connfd = accept(fd, (struct sockaddr *)&client_addr, &socklen);
        if (connfd < 0) {
            continue;   // error
        }

        do_something(connfd);
        close(connfd);
    }

    return 0;
}
```

```cpp
// client.cpp

#include <stdint.h>
#include <stdlib.h>
#include <string.h>
#include <stdio.h>
#include <errno.h>
#include <unistd.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <netinet/ip.h>


static void die(const char *msg) {
    int err = errno;
    fprintf(stderr, "[%d] %s\n", err, msg);
    abort();
}

int main() {
    int fd = socket(AF_INET, SOCK_STREAM, 0);
    if (fd < 0) {
        die("socket()");
    }

    struct sockaddr_in addr = {};
    addr.sin_family = AF_INET;
    addr.sin_port = ntohs(1234);
    addr.sin_addr.s_addr = ntohl(INADDR_LOOPBACK);  // 127.0.0.1
    int rv = connect(fd, (const struct sockaddr *)&addr, sizeof(addr));
    if (rv) {
        die("connect");
    }

    char msg[] = "hello";
    write(fd, msg, strlen(msg));

    char rbuf[64] = {};
    ssize_t n = read(fd, rbuf, sizeof(rbuf) - 1);
    if (n < 0) {
        die("read");
    }
    printf("server says: %s\n", rbuf);
    close(fd);
    return 0;
}
```