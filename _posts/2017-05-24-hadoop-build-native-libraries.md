---
title: "How to build hadoop native libraries"
categories:
  - howto
tags:
  - hadoop
  - lz4
  - redhat
---

For Hadoop version which is used (see below) only GZ compression is available. The task is to enable LZ4 compression and for that native libraries should be compiled.

High level overview how to build native libraries available [here](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/NativeLibraries.html).

## Step by step guide
1. Install required packages:
```
sudo yum install libtool autoconf zlib-devel openssl-devel gcc gcc-c++ cmake
```
2. Define environment variables:
```
cat >>.bashrc <<EOF
export JAVA_HOME=$HOME/java
export HADOOP_HOME=$HOME/hadoop
export HADOOP_PREFIX=$HADOOP_HOME
export HBASE_HOME=$HOME/hbase
export BUILD_ROOT=$HOME/sources
export PATH="$JAVA_HOME/bin:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$HBASE_HOME/bin:$PATH"
EOF
cd $HOME
[ -d BUILD_ROOT ] || mkdir -p $BUILD_ROOT
```
3. Download maven:
```
cd $BUILD_ROOT
wget https://archive.apache.org/dist/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz && \
tar xzvf apache-maven-3.3.9-bin.tar.gz && \
rm -rf apache-maven-3.3.9-bin.tar.gz && \
ln -s apache-maven-3.3.9 maven
export MAVEN_HOME=$BUILD_ROOT/maven
export PATH="$MAVEN_HOME/bin:$PATH"
```
4. Install protocol buffers:
```
cd $BUILD_ROOT
wget https://github.com/google/protobuf/releases/download/v2.5.0/protobuf-2.5.0.tar.gz && \
tar xzvf protobuf-2.5.0.tar.gz && \
rm -rf protobuf-2.5.0.tar.gz
cd protobuf-2.5.0
./configure
make
make check
sudo make install
```
5. Build hadoop native libraries:
```
cd $BUILD_ROOT
git clone https://github.com/apache/hadoop.git
cd hadoop
git checkout branch-2.6.0
mvn package -Pdist,native -Dmaven.javadoc.skip=true -DskipTests -Dtar
```

6. Check for results:
```
LD_LIBRARY_PATH="$LD_LIBRARY_PATH:$HOME/hadoop/lib/native" hadoop checknative -a
hbase org.apache.hadoop.util.NativeLibraryChecker
```
Output should contain `lz4: true` string.
