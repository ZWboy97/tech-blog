---
title: 高性能服务器：实现简单的TCP/UDP server，以及对应的Client
categories: [技术]
tags:
  - linux
  - 高性能服务器
date: 2019-11-20 19:57:20
---

> 有兴趣搞一搞高性能流媒体服务器，这些得了解，学习过程中记录一下，方便未来回顾。

### 一、实现一个TCP Server
##### Server端步骤
- socker():创建socket，指定使用TCP协议
- bind():将socket与地址和端口进行绑定
- listen():侦听端口
- accept():创建新的socket
- recv():使用recv接收数据
- send():使用send发送数据
- close():使用close关闭连接

##### TCP常见套接字选项
- SO_REUSEADDR: 地址重用
    - 之前程序结束，端口处于WAIT_TIME状态下，新启动的程序仍然可以启动
- SO——RCVBUF：设置接收缓冲区大小
- SO_SNDBUF: 设置发送缓冲区大小

##### Demo
```c++
#include <iostream>
#include <sys/socket.h> // for socket(),setsockopt
#include <netinet/in.h> // for struct sockaddr_in
#include <stdlib.h>     // for exit()
#include <unistd.h>     // for close()
#include <string.h>     // for bzero()
#define PORT 8881
#define MESSAGE_LEN 1024
int main(int argc, char *argv[])
{
    int ret = -1;
    int on = 1;
    int socket_fd, accept_fd;
    int backlog = 10; //缓冲长度，与并发量相关
    struct sockaddr_in localaddr, remoteaddr;
    char in_buff[MESSAGE_LEN] = {
        0,
    }; // 接收缓冲区
    socket_fd = socket(PF_INET, SOCK_STREAM, 0);
    if (socket_fd == -1)
    {
        std::cout << "failed to create socket!" << std::endl;
        exit(-1);
    }
    ret = setsockopt(socket_fd, SOL_SOCKET, SO_REUSEADDR,
                     &on, sizeof(on));
    if (ret == -1)
    {
        std::cout << "failed to set socket options!" << std::endl;
    }
    localaddr.sin_family = AF_INET;
    localaddr.sin_port = PORT;
    localaddr.sin_addr.s_addr = INADDR_ANY;
    bzero(&(localaddr.sin_zero), 8);
    ret = bind(socket_fd, (struct sockaddr *)&localaddr, sizeof(struct sockaddr));
    if (ret == -1)
    {
        std::cout << "failed to bind " << std::endl;
        exit(-1);
    }
    ret = listen(socket_fd, backlog);
    if (ret == -1)
    {
        std::cout << "failed to listen " << std::endl;
        exit(-1);
    }
    for (;;)
    {
        socklen_t addr_len = sizeof(struct sockaddr);
        accept_fd = accept(socket_fd,
                           (struct sockaddr *)&remoteaddr,
                           &addr_len);
        int pid = fork();
        for (;;)
        {
            ret = recv(accept_fd, (void *)in_buff, MESSAGE_LEN, 0);
            if (ret == 0) // 说明没数据了
            {
                std::cout << "recv finish，end！ " << std::endl;
                break;
            }
            std::cout << "receive:" << in_buff << std::endl;
            // 返回客户端
            send(accept_fd, (void *)in_buff, MESSAGE_LEN, 0);
        }
        close(accept_fd);
    }
    close(socket_fd);
    return 0;
}
```

### 二、实现一个TCP Client
##### Client端步骤
- socket():创建socket，指定使用TCP协议
- connect():操作系统随机分配一个随机的端口和IP地址
- send():发送
- recv():接收
- close():关闭

##### Demo
```c++
#include <iostream>
#include <sys/socket.h>  //for socket()
#include <sys/types.h>   // for connect()
#include <netinet/in.h>  // for struct sockaddr_in
#include <arpa/inet.h>   // for inet_addr()
#include <stdio.h>       // for gets()
#include <string.h>      // for strlen()
#include <stdlib.h>      // for exit()
#define PORT 8881        // tcp server port
#define MESSAGE_LEN 1024 // buffer size
int main(int argc, char *argv[])
{
    int socket_fd; // file descriptor
    int ret = -1;
    // step1:create socket
    socket_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (socket_fd < 0)
    {
        std::cout << "failed to create socket" << std::endl;
        exit(-1);
    }
    // step2: connect
    struct sockaddr_in serveraddr;
    serveraddr.sin_family = AF_INET;
    serveraddr.sin_port = PORT;
    serveraddr.sin_addr.s_addr = inet_addr("127.0.0.1");
    ret = connect(socket_fd, (struct sockaddr *)&serveraddr, sizeof(struct sockaddr));
    if (ret < 0)
    {
        std::cout << "failed to connect server" << std::endl;
        exit(-1);
    }
    // step3: send()
    char sendbuf[MESSAGE_LEN] = {0};
    char recvbuf[MESSAGE_LEN] = {0};
    while (1)
    {
        memset(sendbuf, 0, MESSAGE_LEN); // 清空buf
        gets(sendbuf);                   // input from console
        ret = send(socket_fd, sendbuf, strlen(sendbuf), 0);
        if (ret <= 0) // 无发送数据
        {
            std::cout << "failed to send data!" << std::endl;
            break;
        }
        if (strcmp(sendbuf, "q") == 0) // input q, quit
        {
            break;
        }
        ret = recv(socket_fd, recvbuf, MESSAGE_LEN, 0);
        recvbuf[ret] = '\0'; //末尾加\0转化为字符串
        std::cout << "receive:" << recvbuf << std::endl;
    }
    return 0;
}
```

### 三、 实现UDPServer
##### Server 流程
- Socket():创建Socket，指定为UDP协议
- bind():将socket与地址和端口绑定
- 不需要listen，因为udp是无连接的
- recvfrom():使用recv/send 
- sendto(): 发送
- close(): 关闭连接

##### demo
```c++
#include <iostream>
#include <sys/socket.h>
#include <netinet/in.h> // for struct sockaddr_in
#include <arpa/inet.h>  // for inet_addr()
#include <stdlib.h>
#define BUFFER_SIZE 1024
int main(int argc, char *argv[])
{
    std::cout << "Welcome! This is a UDP server." << std::endl;

    int ret = -1;

    int socket_fd;
    // step1: create socket
    socket_fd = socket(AF_INET, SOCK_DGRAM, 0);
    if (socket_fd < 0)
    {
        std::cout << "create udp socket failed!" << std::endl;
        exit(-1);
    }
    // step: bind
    struct sockaddr_in addr;
    addr.sin_family = AF_INET; // IPV4
    addr.sin_port = 9876;
    addr.sin_addr.s_addr = inet_addr("127.0.0.1");
    ret = bind(socket_fd, (sockaddr *)&addr, sizeof(struct sockaddr));
    if (ret < 0)
    {
        std::cout << "bind addr and port failed!" << std::endl;
        exit(-1);
    }
    // step3: recvfrom
    struct sockaddr_in clientAddr;
    socklen_t len = sizeof(clientAddr);
    int n;
    char recv_buf[BUFFER_SIZE] = {0};
    char send_buf[BUFFER_SIZE] = {0};

    while (1)
    {
        // receive data to recv_buf
        n = recvfrom(socket_fd, recv_buf, BUFFER_SIZE, 0, (struct sockaddr *)&clientAddr, &len);
        if (n > 0)
        {
            recv_buf[n] = 0; //转为字符串
            std::cout << "receive data:" << recv_buf << std::endl;
            n = sendto(socket_fd, send_buf, BUFFER_SIZE, 0, (struct sockaddr *)&clientAddr, sizeof(clientAddr));
            if (n < 0)
            {
                std::cout << "send error!" << std::endl;
            }
        }
        else
        {
            std::cout << "receive error!" << std::endl;
        }
    }
    return 0;
}
```

### 四、 实现UDP Client
##### Client 流程
- Socket():创建Socket，指定为UDP协议
- sendto(): 发送
- recvfrom():使用recv/send 
- close(): 关闭连接

##### demo
```c++
#include <iostream>
#include <sys/socket.h> // for socket()
#include <stdlib.h>     // for exit()
#include <netinet/in.h> // for struct sockaddr_in
#include <arpa/inet.h>  // for inet_addr()
#include <string.h>     // for strlen
#include <unistd.h>     // for close()
#define BUFFER_SIZE 1024
int main(int argc, char *argv[])
{
    // step1: socket
    int socket_fq;
    socket_fq = socket(AF_INET, SOCK_DGRAM, 0); // UDP是基于报文的
    if (socket_fq < 0)
    {
        std::cout << "create socket failed!" << std::endl;
        exit(-1);
    }
    // step2:
    struct sockaddr_in serverAddr;
    serverAddr.sin_family = AF_INET;
    serverAddr.sin_port = 9876;
    serverAddr.sin_addr.s_addr = inet_addr("127.0.0.1");
    char send_buf[BUFFER_SIZE] = "hello";
    char recv_buf[BUFFER_SIZE] = {0};
    socklen_t len = sizeof(serverAddr);
    int n = 0;
    if (serverAddr.sin_addr.s_addr == INADDR_NONE)
    {
        std::cout << "Incorrect ip address!" << std::endl;
        close(socket_fq);
        exit(-1);
    }
    n = sendto(socket_fq, send_buf, strlen(send_buf), 0, (sockaddr *)&serverAddr, sizeof(serverAddr));
    if (n < 0)
    {
        std::cout << "send error!" << std::endl;
        close(socket_fq);
    }
    n = recvfrom(socket_fq, recv_buf, BUFFER_SIZE, 0, (sockaddr *)&serverAddr, &len);
    if (n > 0)
    {
        recv_buf[n] = 0;
        std::cout << "receive:" << recv_buf << std::endl;
    }
    else if (n == 0)
    {
        std::cout << "server closed." << std::endl;
    }
    else if (n == -1)
    {
        std::cout << "recvfrom error." << std::endl;
    }
    close(socket_fq);
    return 0;
}
```

### 总结
哎呦，不错哦；