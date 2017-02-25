*Last updated on 25th Feb 2017*

Instructions to build ClickHouse on Centos 7

The folks over at ClickHouse have some instructions to build binaries for Debian [here](https://github.com/yandex/ClickHouse/blob/master/doc/build.md). If you need build a rpm, you can follow the steps mentioned [here](https://github.com/redsoftbiz/clickhouse-rpm).

This note is meant for those who simply need to get ClickHouse running on Centos 7 without the explicit need to get rpms in place. I am going to try and map these instructions to those mentioned by the ClickHouse build instructions (wherever possible).

**Test for SSE 4.2**
**Install gcc and gcc-c++**
**Install Git and CMake**

....
