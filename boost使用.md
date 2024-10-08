# Boost库

## 安装Boost库 [Linux]

---

### 下载

- 地址：https://archives.boost.io/release/

### 解压

```sh
tar -xzvf boost_1_82_0.tar.gz
cd boost_1_82_0
```

 ### 编译

```shell
./bootstrap.sh --prefix=/usr/local
./b2
```



### 安装

```shell
sudo ./b2 install
```



### 编译

```shell
g++ -o boost_tcp_client ./boost_tcpserver.c -lpthread -lboost_system

g++ -o boost_tcp_client ./boost_tcpserver.c -lpthread -lboost_system
```



## Boost库 使用[Linux]

---

> 官网：`https://www.boost.org/doc/libs/1_86_0/` 

### Boost算法

### Boost字符处理

##### boost_character.c

```cpp
#include <iostream>
#include <string>
#include "boost/algorithm/string.hpp"
#include "boost/algorithm/hex.hpp"

int main()
{
    // low 2 up
    std::string str = "lvs string 1";
    boost::to_upper(str);
    std::cout <<"to_upper:"<< str<<std::endl;

    // Up 2 Low
    std::string str2 = "LVS STRING 2";
    boost::to_lower(str2);
    std::cout <<"to_lower:"<< str2<<std::endl;

    // find characters in string
    std::string str3 = "LVS STRING ,here";
    if(boost::find_first(str3,"here")){
        std::cout <<"here found"<<std::endl;
    }

    // replace characters in string
    std::string str4 = "LVS STRING ,here replace";
    boost::replace_all(str4,"replace","lv lv lv ");
    std::cout<<"Replaced string:"<<str4<<std::endl;

    // character 2 hex
    std::string input = "hello,boost";
    std::string hex_output,unhex_output;

    boost::algorithm::hex(input.begin(),input.end(),std::back_inserter(hex_output));
    std::cout << "hex_output:"<< hex_output << std::endl;

    // hex 2 character string
    boost::algorithm::unhex(hex_output.begin(),hex_output.end(),std::back_inserter(unhex_output));
    std::cout << "unhex_output:"<< unhex_output << std::endl;

    return 0;
}


```



##### 编译

```makefile
BOOSTDIR := /home/lvs/lvs/07_ThirdCompontents/03_boost/01_release_build/
SYSDIR := /usr/lib/
CURREENT_PATH := $(shell pwd)
CXX = g++
CXXFLAGS = -std=c++17

Target = boost_character

.PHONY: all

$(Target):
	$(CXX) -o $(Target) $(CURREENT_PATH)/$(Target).c -I$(BOOSTDIR)include -L$(BOOSTDIR)lib -L$(SYSDIR) -lboost_system 

clean:
	rm -f $(Target)
	echo "clean ok"

all:$(Target)
```



##### 运行

```shell
lvs@ubuntu:~/lvs/01_Code/01_TestCode01/BoostTest/BoostCharacter$ ./boost_character 
to_upper:LVS STRING 1
to_lower:lvs string 2
here found
Replaced string:LVS STRING ,here lv lv lv 
hex_output:68656C6C6F2C626F6F7374
unhex_output:hello,boost
```



### Boost时间日期

### Boost文件处理

### Boost环形缓冲区

#### boost_circular_buffer.c

```cpp
#include <iostream>
#include <string>
#include "boost/circular_buffer.hpp"

int main(int argc,char** argv)
{
    boost::circular_buffer<std::string> circluar_buffer(5);
    for(int i = 0 ;i < 10;i++){
        circluar_buffer.push_back(std::to_string(i));
    }

    int _size = circluar_buffer.size();
    for(int i = 0 ;i < _size;i++){
        std::cout <<circluar_buffer[i]<<std::endl;
    }

    return 0;
}
```

##### 编译

```makefile
BOOSTDIR := /home/lvs/lvs/07_ThirdCompontents/03_boost/01_release_build/
SYSDIR := /usr/lib/
CURREENT_PATH := $(shell pwd)
CXX = g++
CXXFLAGS = -std=c++17

Target = boost_circular_buffer

.PHONY: all

$(Target):
	$(CXX) -o $(Target) $(CURREENT_PATH)/$(Target).c -I$(BOOSTDIR)include -L$(BOOSTDIR)lib -L$(SYSDIR) -lboost_system 

clean:
	rm -f $(Target)
	echo "clean ok"

all:$(Target)
```



##### 运行

```shell
lvs@ubuntu:~/lvs/01_Code/01_TestCode01/BoostTest/BoostCircularBuffer$ ./boost_circular_buffer 
5
6
7
8
9
```



### Boost容器篇

### Boost智能指针

### Boost线程

### Boost Lambda

### Boost ASIO

#### Asio_tcp

> 线程使用std::thread

##### boost_tcpclient.c

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

##### boost_tcpserver.c

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

##### 编译

1. 命令行编译

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

2. make编译

```makefile
BOOSTDIR := /home/lvs/lvs/07_ThirdCompontents/03_boost/01_release_build/
CURREENT_PATH := $(shell pwd)
CXX = g++
CXXFLAGS = -std=c++17

.PHONY: all

boost_tcp_server:
	$(CXX) -o boost_tcp_server $(CURREENT_PATH)/boost_tcpserver.c -I$(BOOSTDIR)include -L$(BOOSTDIR)lib -L/usr/lib/ -lpthread -lboost_system 

boost_tcp_client:
	$(CXX) -o boost_tcp_client $(CURREENT_PATH)/boost_tcpclient.c -I$(BOOSTDIR)include -L$(BOOSTDIR)lib -L/usr/lib/ -lpthread -lboost_system 

clean:
	rm -f boost_tcp_server boost_tcp_client
	echo "clean ok"

all:boost_tcp_server boost_tcp_client
```

