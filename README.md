# p4-sample

[P4 & XDP（VxLANの中身でフィルタリングしてみる）](https://qiita.com/hibitomo/items/3ed846beb2e504f0ffb6)
を写経してみる。

## p4c-xdpのインストール

記事ではUbuntu18.04とのことだが、手元にあった14.04で試してみる。

```sh
❯ vagrant ssh
Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 3.13.0-161-generic x86_64)
```

```sh
$ sudo apt update && sudo apt upgrade -y
$ sudo apt install cmake g++ git automake libtool libgc-dev bison flex libfl-dev libgmp-dev libboost-dev libboost-iostreams-dev libboost-graph-dev llvm pkg-config python python-scapy python-ipaddr python-ply tcpdump python-pip unzip
$ sudo pip install tenjin pyroute2 ply scapy
$ sudo apt install clang llvm libpcap-dev libelf-dev iproute2 net-tools
$ wget http://github.com/protocolbuffers/protobuf/releases/download/v3.6.1/protobuf-cpp-3.6.1.zip
$ unzip protobuf-cpp-3.6.1.zip
$ cd protobuf-3.6.1
$ ./configure
$ make
$ make check
$ sudo make install
$ sudo ldconfig
```

```sh
$ git clone https://github.com/p4lang/p4c
$ cd p4c
$ git checkout 027098cc5720f7a03f5b7e745d15e5d840c8daa0
$ git submodule update --init --recursive
$ mkdir -p extensions
$ cd extensions
$ git clone https://github.com/vmware/p4c-xdp
$ ln -s ~/p4c p4c-xdp/p4c
$ cd ..
$ mkdir -p build
$ cd build
$ cmake .. '-DCMAKE_CXX_FLAGS:STRING=-O2'
```


```sh
$ cmake .. '-DCMAKE_CXX_FLAGS:STRING=-O2'
CMake Error at CMakeLists.txt:15 (cmake_minimum_required):
  CMake 3.0.2 or higher is required.  You are running version 2.8.12.2


-- Configuring incomplete, errors occurred!
```

cmakeの最新版をインストールする。

```sh
$ cd ~
$ https://github.com/Kitware/CMake/releases/download/v3.13.2/cmake-3.13.2.tar.gz
$ tar xvf cmake-3.13.2.tar.gz
$ cd cmake-3.13.2
$ ./bootstrap && make && make install
```

最後の最後でこれが出た。（`make`のあとだから？）

```sh
-- Install configuration: ""
CMake Error at cmake_install.cmake:41 (file):
  file cannot create directory: /usr/local/doc/cmake-3.13.  Maybe need
  administrative privileges.


make: *** [install] Error 1
```

```sh
$ ./bootstrap && make && make install
```

```sh
$ cd ~/protobuf-3.6.1/p4c/build/
```

```sh
$ cmake .. '-DCMAKE_CXX_FLAGS:STRING=-O2'
CMake Error: Could not find CMAKE_ROOT !!!
CMake has most likely not been installed correctly.
Modules directory not found in
/usr/local/bin
CMake Error: Error executing cmake::LoadCache(). Aborting.
```

もう一回戻ってやりなおす。

```sh
$ cd ~/protobuf-3.6.1
$ ./configure
$ make
$ make check
```

```sh
$ make check
Making check in .
make[1]: Entering directory `/home/vagrant/protobuf-3.6.1'
make  check-local
make[2]: Entering directory `/home/vagrant/protobuf-3.6.1'
Making lib/libgmock.a lib/libgmock_main.a in gmock
make[3]: Entering directory `/home/vagrant/protobuf-3.6.1/third_party/googletest/googletest'
make[3]: `lib/libgtest.la' is up to date.
make[3]: `lib/libgtest_main.la' is up to date.
make[3]: Leaving directory `/home/vagrant/protobuf-3.6.1/third_party/googletest/googletest'
make[3]: Entering directory `/home/vagrant/protobuf-3.6.1/third_party/googletest/googlemock'
make[3]: `lib/libgmock.la' is up to date.
make[3]: `lib/libgmock_main.la' is up to date.
make[3]: Leaving directory `/home/vagrant/protobuf-3.6.1/third_party/googletest/googlemock'
make[2]: Leaving directory `/home/vagrant/protobuf-3.6.1'
make[1]: Leaving directory `/home/vagrant/protobuf-3.6.1'
Making check in src
make[1]: Entering directory `/home/vagrant/protobuf-3.6.1/src'
make  protoc protobuf-test protobuf-lazy-descriptor-test protobuf-lite-test test_plugin protobuf-lite-arena-test no-warning-test
make[2]: Entering directory `/home/vagrant/protobuf-3.6.1/src'
make[2]: `protoc' is up to date.
g++ -DHAVE_CONFIG_H -I. -I..  -I./../third_party/googletest/googletest/include -I./../third_party/googletest/googlemock/include  -pthread -DHAVE_PTHREAD=1  -Wall -Wno-sign-compare -g -std=c++11 -DNDEBUG -MT google/protobuf/protobuf_test-descriptor_unittest.o -MD -MP -MF google/protobuf/.deps/protobuf_test-descriptor_unittest.Tpo -c -o google/protobuf/protobuf_test-descriptor_unittest.o `test -f 'google/protobuf/descriptor_unittest.cc' || echo './'`google/protobuf/descriptor_unittest.cc
In file included from ./../third_party/googletest/googletest/include/gtest/gtest.h:1872:0,
                 from ./../third_party/googletest/googlemock/include/gmock/internal/gmock-internal-utils.h:47,
                 from ./../third_party/googletest/googlemock/include/gmock/gmock-actions.h:46,
                 from ./../third_party/googletest/googlemock/include/gmock/gmock.h:58,
                 from ./google/protobuf/testing/googletest.h:40,
                 from google/protobuf/descriptor_unittest.cc:60:
google/protobuf/descriptor_unittest.cc: In member function ‘virtual void google::protobuf::descriptor_unittest::CustomOptions_OptionTypes_Test::TestBody()’:
./../third_party/googletest/googletest/include/gtest/internal/gtest-internal.h:133:55: warning: converting ‘false’ to pointer type for argument 1 of ‘char testing::internal::IsNullLiteralHelper(testing::internal::Secret*)’ [-Wconversion-null]
     (sizeof(::testing::internal::IsNullLiteralHelper(x)) == 1)
                                                       ^
./../third_party/googletest/googletest/include/gtest/gtest_pred_impl.h:77:52: note: in definition of macro ‘GTEST_ASSERT_’
   if (const ::testing::AssertionResult gtest_ar = (expression)) \
                                                    ^
./../third_party/googletest/googletest/include/gtest/gtest_pred_impl.h:162:3: note: in expansion of macro ‘GTEST_PRED_FORMAT2_’
   GTEST_PRED_FORMAT2_(pred_format, v1, v2, GTEST_NONFATAL_FAILURE_)
   ^
./../third_party/googletest/googletest/include/gtest/gtest.h:1921:3: note: in expansion of macro ‘EXPECT_PRED_FORMAT2’
   EXPECT_PRED_FORMAT2(::testing::internal:: \
   ^
./../third_party/googletest/googletest/include/gtest/gtest.h:1922:32: note: in expansion of macro ‘GTEST_IS_NULL_LITERAL_’
                       EqHelper<GTEST_IS_NULL_LITERAL_(val1)>::Compare, \
                                ^
google/protobuf/descriptor_unittest.cc:2958:3: note: in expansion of macro ‘EXPECT_EQ’
   EXPECT_EQ(false    , options->GetExtension(protobuf_unittest::bool_opt));
   ^
g++: internal compiler error: Killed (program cc1plus)
Please submit a full bug report,
with preprocessed source if appropriate.
See <file:///usr/share/doc/gcc-4.8/README.Bugs> for instructions.
make[2]: *** [google/protobuf/protobuf_test-descriptor_unittest.o] Error 4
make[2]: Leaving directory `/home/vagrant/protobuf-3.6.1/src'
make[1]: *** [check-am] Error 2
make[1]: Leaving directory `/home/vagrant/protobuf-3.6.1/src'
make: *** [check-recursive] Error 1
```

無視して次のコマンド叩いてみる。

```sh
$ sudo make install
$ sudo ldconfig
```

```sh
$ cd ~/protobuf-3.6.1/p4c/build/
$ cmake .. '-DCMAKE_CXX_FLAGS:STRING=-O2'
```

同じエラー(`CMake Error: Could not find CMAKE_ROOT !!!`)が出てる。

`cmake --version`も叩けないので、インストールできていないっっぽい。

[\[Linux\]\[cmake\] 最新のcmakeをインストールする方法](https://qiita.com/koara-local/items/9d01c6bb9dd93563b7c6)を参考にcmakeをインストールしてみる。

```sh
$ wget https://cmake.org/files/v3.6/cmake-3.6.2.tar.gz
$ tar xvf cmake-3.6.2.tar.gz
$ ./bootstrap && make && sudo make install
```

```sh
$ cmake --version
CMake Error: Could not find CMAKE_ROOT !!!
CMake has most likely not been installed correctly.
Modules directory not found in
/usr/local/bin
Segmentation fault (core dumped)
```

```sh
$ /usr/local/bin/cmake --version
cmake version 3.6.2

CMake suite maintained and supported by Kitware (kitware.com/cmake).
```

明示的にこっちのcmakeを叩けばよさそう。

```sh
$ cd ~/protobuf-3.6.1/p4c/build/
$ /usr/local/bin/cmake .. '-DCMAKE_CXX_FLAGS:STRING=-O2'
```

なんとか良さそうのなので、続けていく。


```sh
$ make
[ 12%] Building CXX object ir/CMakeFiles/ir.dir/unified_ir_srcs_1.cpp.o
c++: internal compiler error: Killed (program cc1plus)
Please submit a full bug report,
with preprocessed source if appropriate.
See <file:///usr/share/doc/gcc-4.8/README.Bugs> for instructions.
make[2]: *** [ir/CMakeFiles/ir.dir/unified_ir_srcs_1.cpp.o] Error 4
make[1]: *** [ir/CMakeFiles/ir.dir/all] Error 2
make: *** [all] Error 2
```

メモリが足りないみたいなので、Vagrantfileに以下を追記する。
※コメントアウトを外してメモリを`2048`にした。

```Vagrantfile
config.vm.provider "virtualbox" do |vb|
#   # Customize the amount of memory on the VM:
  vb.memory = "2048"
end
```

```sh
❯ vagrant reload
❯ vagrant ssh
```

## References
* [P4 & XDP（VxLANの中身でフィルタリングしてみる）](https://qiita.com/hibitomo/items/3ed846beb2e504f0ffb6)
* [\[Linux\]\[cmake\] 最新のcmakeをインストールする方法](https://qiita.com/koara-local/items/9d01c6bb9dd93563b7c6)
* [Vagrant + Chef + CentOS6.5 でOpenCV2.4.9をインストールする。](https://qiita.com/arakaji/items/b29e5b8fd479a336bf6b)