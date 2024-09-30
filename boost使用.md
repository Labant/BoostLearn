# Boost库

## 安装Boost库 [Linux]

1. 下载

   - 地址：https://archives.boost.io/release/

2. 解压

   ```sh
   tar -xzvf boost_1_82_0.tar.gz
   cd boost_1_82_0
   ```

3. 编译

   ```shell
   ./bootstrap.sh --prefix=/usr/local
   ./b2
   ```

   

4. 安装

   ```shell
   sudo ./b2 install
   ```

   

5. 使用

```shell
g++ -o boost_tcp_client ./boost_tcpserver.c -lpthread -lboost_system

g++ -o boost_tcp_client ./boost_tcpserver.c -lpthread -lboost_system
```

## Boost网络

### Asio_tcp

> 线程使用std::thread

#### boost_tcpclient.c

```cpp
#include <iostream>
#include <boost/asio.hpp>

using boost::asio::ip::tcp;

int main() {
    try {
        boost::asio::io_context io_context;
        tcp::resolver resolver(io_context);
        tcp::resolver::results_type endpoints = resolver.resolve("127.0.0.1", "12345");

        tcp::socket socket(io_context);
        boost::asio::connect(socket, endpoints);

        for (;;) {
            std::cout << "Enter message: ";
            std::string message;
            std::getline(std::cin, message);

            boost::asio::write(socket, boost::asio::buffer(message));

            char reply[1024];
            size_t reply_length = boost::asio::read(socket, boost::asio::buffer(reply, message.size()));
            std::cout << "Reply is: ";
            std::cout.write(reply, reply_length);
            std::cout << "\n";
        }
    } catch (std::exception& e) {
        std::cerr << "Exception: " << e.what() << "\n";
    }
    return 0;
}

```

#### boost_tcpserver.c

```cpp
#include <iostream>
#include <boost/asio.hpp>

using boost::asio::ip::tcp;

void session(tcp::socket sock) {
    try {
        for (;;) {
            char data[1024];
            boost::system::error_code error;
            size_t length = sock.read_some(boost::asio::buffer(data), error);
            if (error == boost::asio::error::eof)
                break; // 连接关闭
            else if (error)
                throw boost::system::system_error(error); // 其他错误

            std::cout << "[Server]Enter message: ";
            std::string message;
            std::getline(std::cin, message);

            boost::asio::write(sock, boost::asio::buffer(message));
        }
    } catch (std::exception& e) {
        std::cerr << "Exception in thread: " << e.what() << "\n";
    }
}

void server(boost::asio::io_context& io_context, short port) {
    tcp::acceptor acceptor(io_context, tcp::endpoint(tcp::v4(), port));
    for (;;) {
        tcp::socket socket(io_context);
        acceptor.accept(socket);
        std::thread(session, std::move(socket)).detach();
    }
}

int main() {
    try {
        boost::asio::io_context io_context;
        server(io_context, 12345);
    } catch (std::exception& e) {
        std::cerr << "Exception: " << e.what() << "\n";
    }
    return 0;
}

```

#### 编译

```shell
g++ -o boost_tcp_client ./boost_tcpserver.c \
-I/home/lvs/lvs/07_ThirdCompontents/03_boost/01_release_build/include  \
-L/home/lvs/lvs/07_ThirdCompontents/03_boost/01_release_build/lib \
-lpthread \
-lboost_system

g++ -o boost_tcp_client ./boost_tcpserver.c \
-I/home/lvs/lvs/07_ThirdCompontents/03_boost/01_release_build/include  \
-L/home/lvs/lvs/07_ThirdCompontents/03_boost/01_release_build/lib \
-lpthread \
-lboost_system
```

