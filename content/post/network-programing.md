---
title: "网络编程-C++"
date: 2020-07-01T15:09:10+08:00
draft: false
tags: 
- cpp
- 网络
categories: 
- 学习笔记
---

# Socket编程

## 服务端

### TCP

- socket()  创建socket
- bind() 分配socket地址
- listen() 监听socket
- accpet() 接收连接请求
- recv()/send() 数据交换
- close() 关闭连接

### UDP

- socket()  创建socket
- bind() 分配socket地址
- recvfrom()/sendto() 数据交换
- close() 关闭连接

## 客户端

### TCP

- socket()  创建socket
- connect() 连接服务器
- recv()/send() 数据交换
- close() 关闭连接

### UDP

- socket()  创建socket
- recvfrom()/sendto() 数据交换
- close() 关闭连接

## 实现

### tcp server

``` cpp
#include <cstring>
#include <iostream>
#include <netinet/in.h>
#include <sys/socket.h>
#include <unistd.h>

#define PORT 2333        //service port
#define MESSAGE_LEN 1024 //
#define FD_SIZE 1024

int main()
{
    int sockfd = -1, accpetfd = -1;           // socket file descriptor
    char in_buffer[MESSAGE_LEN];              //buffer
    int ret = -1;                             //return value
    int reuse = 1;                            //set socket reuse address option value
    struct sockaddr_in localaddr, remoteaddr; //store some address
    int backlog = 10;                         //
    sockfd = socket(AF_INET,                  //ipv4
                    SOCK_STREAM,              //tcp
                    0);
    if (sockfd == -1)
    {
        std::cout << "Failed to create socket!" << std::endl;
        exit(EXIT_FAILURE);
    }
    //int setsockopt(int sockfd, int level, int optname,
    //               const void *optval, socklen_t optlen);
    if (setsockopt(sockfd,
                   SOL_SOCKET,           //socket level
                   SO_REUSEADDR,         //reuse address
                   &reuse,               //set reuse address on
                   sizeof(reuse)) == -1) //option value length
    {
        std::cout << "Failed to set socket options!" << std::endl;
        exit(EXIT_FAILURE);
    }

    localaddr.sin_family = AF_INET;         // ipv4
    localaddr.sin_port = htons(PORT);       //bind port
    localaddr.sin_addr.s_addr = INADDR_ANY; //bind any address
    bzero(localaddr.sin_zero, sizeof(localaddr.sin_zero));

    if (bind(sockfd, (const sockaddr *)&localaddr, sizeof(localaddr)) < 0)
    {
        std::cout << "Failed to bind address!" << std::endl;
        exit(EXIT_FAILURE);
    }

    if (listen(sockfd, backlog) < 0)
    {
        std::cout << "Failed to listen!" << std::endl;
        exit(EXIT_FAILURE);
    }
    for (;;)
    {
        socklen_t addrlen = sizeof(struct sockaddr);
        accpetfd = accept(sockfd,
                          (struct sockaddr *)&remoteaddr,
                          &addrlen);
        for (;;)
        {
            memset(in_buffer, 0, sizeof(in_buffer));
            ret = recv(accpetfd, in_buffer, MESSAGE_LEN, 0);
            if (ret == 0)
            {
                break;
            }
            std::cout << "recv: " << in_buffer << std::endl;
            send(accpetfd, in_buffer, MESSAGE_LEN, 0);
        }

        close(accpetfd);
    }
    close(sockfd);
    return 0;
}
```

 ### tcp server with fork

```cpp
#include <cstring>
#include <iostream>
#include <netinet/in.h>
#include <sys/socket.h>
#include <unistd.h>

#define PORT 2333        //service port
#define MESSAGE_LEN 1024 //
#define FD_SIZE 1024

int main()
{
    int sockfd = -1, accpetfd = -1;           // socket file descriptor
    char in_buffer[MESSAGE_LEN];              //buffer
    int ret = -1;                             //return value
    int reuse = 1;                            //set socket reuse address option value
    struct sockaddr_in localaddr, remoteaddr; //store some address
    int backlog = 10;                         //
    pid_t pid = -1;                           //fork pid
    sockfd = socket(AF_INET,                  //ipv4
                    SOCK_STREAM,              //tcp
                    0);
    if (sockfd == -1)
    {
        std::cout << "Failed to create socket!" << std::endl;
        exit(EXIT_FAILURE);
    }
    //int setsockopt(int sockfd, int level, int optname,
    //               const void *optval, socklen_t optlen);
    if (setsockopt(sockfd,
                   SOL_SOCKET,           //socket level
                   SO_REUSEADDR,         //reuse address
                   &reuse,               //set reuse address on
                   sizeof(reuse)) == -1) //option value length
    {
        std::cout << "Failed to set socket options!" << std::endl;
        exit(EXIT_FAILURE);
    }

    localaddr.sin_family = AF_INET;         // ipv4
    localaddr.sin_port = htons(PORT);       //bind port
    localaddr.sin_addr.s_addr = INADDR_ANY; //bind any address
    bzero(localaddr.sin_zero, sizeof(localaddr.sin_zero));

    if (bind(sockfd, (const sockaddr *)&localaddr, sizeof(localaddr)) < 0)
    {
        std::cout << "Failed to bind address!" << std::endl;
        exit(EXIT_FAILURE);
    }

    if (listen(sockfd, backlog) < 0)
    {
        std::cout << "Failed to listen!" << std::endl;
        exit(EXIT_FAILURE);
    }
    for (;;)
    {
        socklen_t addrlen = sizeof(struct sockaddr);
        accpetfd = accept(sockfd,
                          (struct sockaddr *)&remoteaddr,
                          &addrlen);
        pid = fork();
        if (pid == 0)
        {
            for (;;)
            {
                memset(in_buffer, 0, sizeof(in_buffer));
                ret = recv(accpetfd, in_buffer, MESSAGE_LEN, 0);
                if (ret == 0)
                {
                    break;
                }
                std::cout << "recv: " << in_buffer << std::endl;
                send(accpetfd, in_buffer, MESSAGE_LEN, 0);
            }
            close(accpetfd);
        }
    }

    if (pid != 0)
    {
        close(sockfd);
    }
    return 0;
}
```

### tcp server with select

```cpp
#include <cstring>
#include <fcntl.h>
#include <iostream>
#include <netinet/in.h>
#include <sys/socket.h>
#include <unistd.h>

#define PORT 2333        //service port
#define MESSAGE_LEN 1024 //
#define FD_SIZE 1024

int main()
{
    int sockfd = -1, accpetfd = -1;           // socket file descriptor
    char in_buffer[MESSAGE_LEN];              //buffer
    int ret = -1;                             //return value
    int reuse = 1;                            //set socket reuse address option value
    struct sockaddr_in localaddr, remoteaddr; //store some address
    int backlog = 10;                         //
    int flags = 0;                            //socket flags
    int maxfd = 0;                            //max file descriptor
    int events = 0;                           //store select return value
    fd_set fd_sets;                           //file descriptor set
    int accept_fds[FD_SIZE] = {
        -1,
    };               //accept file descriptor
    int curpos = -1; //current position

    sockfd = socket(AF_INET,     //ipv4
                    SOCK_STREAM, //tcp
                    0);
    if (sockfd == -1)
    {
        std::cout << "Failed to create socket!" << std::endl;
        exit(EXIT_FAILURE);
    }
    maxfd = sockfd;

    flags = fcntl(sockfd, F_GETFL, 0);          //get socket flag
    fcntl(sockfd, F_SETFL, flags | O_NONBLOCK); //set socket non-block

    //int setsockopt(int sockfd, int level, int optname,
    //               const void *optval, socklen_t optlen);
    if (setsockopt(sockfd,
                   SOL_SOCKET,           //socket level
                   SO_REUSEADDR,         //reuse address
                   &reuse,               //set reuse address on
                   sizeof(reuse)) == -1) //option value length
    {
        std::cout << "Failed to set socket options!" << std::endl;
        exit(EXIT_FAILURE);
    }

    localaddr.sin_family = AF_INET;                        //ipv4
    localaddr.sin_port = htons(PORT);                      //bind port
    localaddr.sin_addr.s_addr = INADDR_ANY;                //bind any address
    bzero(localaddr.sin_zero, sizeof(localaddr.sin_zero)); //fill zero

    if (bind(sockfd, (const sockaddr *)&localaddr, sizeof(localaddr)) < 0)
    {
        std::cout << "Failed to bind address!" << std::endl;
        exit(EXIT_FAILURE);
    }

    if (listen(sockfd, backlog) < 0)
    {
        std::cout << "Failed to listen!" << std::endl;
        exit(EXIT_FAILURE);
    }
    for (;;)
    {
        FD_ZERO(&fd_sets);        // initial file descriptor set
        FD_SET(sockfd, &fd_sets); // insert sockfd to fd_sets

        for (int i = 0; i < FD_SIZE; i++)
        {
            if (accept_fds[i] != -1)
            {
                if (accept_fds[i] > maxfd)
                {
                    maxfd = accept_fds[i];
                }
                FD_SET(accept_fds[i], &fd_sets);
            }
        }
        events = select(maxfd + 1, &fd_sets, NULL, NULL, NULL);
        if (events < 0)
        {
            std::cout << "Failed to use select!" << std::endl;
            break;
        }
        else if (events == 0)
        {
            std::cout << "timeout..." << std::endl;
            continue;
        }
        else
        {
            if (FD_ISSET(sockfd, &fd_sets))
            {
                for (int i = 0; i < FD_SIZE; i++)
                {
                    if (accept_fds[i] == -1)
                    {
                        curpos = i;
                        break;
                    }
                }
                socklen_t addrlen = sizeof(struct sockaddr);
                accpetfd = accept(sockfd,
                                  (struct sockaddr *)&remoteaddr,
                                  &addrlen);
                flags = fcntl(accpetfd, F_GETFL, 0);
                fcntl(accpetfd, F_SETFL, flags | O_NONBLOCK);
                accept_fds[curpos] = accpetfd;
            }
            for (int i = 0; i < FD_SIZE; i++)
            {
                if (accept_fds[i] != -1 && FD_ISSET(accept_fds[i], &fd_sets))
                {
                    memset(in_buffer, 0, sizeof(in_buffer));
                    ret = recv(accept_fds[i], in_buffer, MESSAGE_LEN, 0);
                    if (ret == 0)
                    {
                        close(accept_fds[i]);
                        break;
                    }
                    std::cout << "recv: " << in_buffer << std::endl;
                    send(accept_fds[i], in_buffer, MESSAGE_LEN, 0);
                }
            }
        }
    }
    close(sockfd);
    return 0;
}
```

### tcp server with epoll

```cpp
#include <cstring>
#include <fcntl.h>
#include <iostream>
#include <netinet/in.h>
#include <sys/epoll.h>
#include <sys/socket.h>
#include <unistd.h>

#define PORT 2333        //service port
#define MESSAGE_LEN 1024 //
#define FD_SIZE 1024
#define MAX_EVENTS 20
#define TIMEOUT 500

int main()
{
    int sockfd = -1, accpetfd = -1;            // socket file descriptor
    char in_buffer[MESSAGE_LEN];               //buffer
    int ret = -1;                              //return value
    int reuse = 1;                             //set socket reuse address option value
    struct sockaddr_in localaddr, remoteaddr;  //store some address
    int backlog = 10;                          //
    int ep_fd = -1;                            //epoll file descriptor
    struct epoll_event ev, events[MAX_EVENTS]; //
    int flags = 0;                             //socket flags
    int event_number = 0;

    sockfd = socket(AF_INET,     //ipv4
                    SOCK_STREAM, //tcp
                    0);

    if (sockfd == -1)
    {
        std::cout << "Failed to create socket!" << std::endl;
        exit(EXIT_FAILURE);
    }

    flags = fcntl(sockfd, F_GETFL, 0);          //get socket flag
    fcntl(sockfd, F_SETFL, flags | O_NONBLOCK); //set socket non-block

    //int setsockopt(int sockfd, int level, int optname,
    //               const void *optval, socklen_t optlen);
    if (setsockopt(sockfd,
                   SOL_SOCKET,           //socket level
                   SO_REUSEADDR,         //reuse address
                   &reuse,               //set reuse address on
                   sizeof(reuse)) == -1) //option value length
    {
        std::cout << "Failed to set socket options!" << std::endl;
        exit(EXIT_FAILURE);
    }

    localaddr.sin_family = AF_INET;         // ipv4
    localaddr.sin_port = htons(PORT);       //bind port
    localaddr.sin_addr.s_addr = INADDR_ANY; //bind any address
    bzero(localaddr.sin_zero, sizeof(localaddr.sin_zero));

    if (bind(sockfd, (const sockaddr *)&localaddr, sizeof(localaddr)) < 0)
    {
        std::cout << "Failed to bind address!" << std::endl;
        exit(EXIT_FAILURE);
    }

    if (listen(sockfd, backlog) < 0)
    {
        std::cout << "Failed to listen!" << std::endl;
        exit(EXIT_FAILURE);
    }

    ep_fd = epoll_create(256);
    ev.events = EPOLLIN;
    ev.data.fd = sockfd;
    epoll_ctl(ep_fd, EPOLL_CTL_ADD, sockfd, &ev);
    for (;;)
    {
        event_number = epoll_wait(ep_fd, events, MAX_EVENTS, TIMEOUT);
        for (int i = 0; i < event_number; i++)
        {
            if (events[i].data.fd == sockfd)
            {
                socklen_t addrlen = sizeof(struct sockaddr);
                accpetfd = accept(sockfd, (struct sockaddr *)&remoteaddr, &addrlen);
                flags = fcntl(accpetfd, F_GETFL, 0);
                fcntl(accpetfd, F_SETFL, flags | O_NONBLOCK);

                ev.events = EPOLLIN | EPOLLET;
                ev.data.fd = accpetfd;
                epoll_ctl(ep_fd, EPOLL_CTL_ADD, accpetfd, &ev);
            }
            else if (events[i].events & EPOLLIN)
            {

                memset(in_buffer, 0, sizeof(in_buffer));
                ret = recv(events[i].data.fd, in_buffer, MESSAGE_LEN, 0);

                if (ret == MESSAGE_LEN)
                {
                    std::cout << "maybe have data..." << std::endl;
                }

                if (ret <= 0)
                {
                    switch (errno)
                    {
                    case EAGAIN:
                        break;
                    case EINTR:
                        std::cout << "recv EINTR" << std::endl;
                        ret = recv(events[i].data.fd, in_buffer, MESSAGE_LEN, 0);
                        break;
                    default:
                        break;
                    }
                }
                std::cout << "receive message: " << in_buffer << std::endl;
                send(events[i].data.fd, in_buffer, MESSAGE_LEN, 0);
            }
        }
    }
    close(sockfd);
    return 0;
}
```

