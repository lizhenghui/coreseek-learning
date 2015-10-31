## Coreseek安装

### 1. 下载
wget http://www.coreseek.cn/uploads/csft/4.0/coreseek-4.1-beta.tar.gz 

### 2. 安装mmseg3
```	
tar zxvf coreseek-4.1-beta.tar.gz    
cd coreseek-4.1-beta  
cd mmseg-3.2.14  
./bootstrap  
./configure --prefix=/usr/local/mmseg3  
make && make install 
```

如果出错请执行如下命令
```
yum -y install libtool  
aclocal  
libtoolize --force  
automake --add-missing  
autoconf  
autoheader  
make clean
```

### 3. 安装coreseek
```
cd ..
yum -y install expat expat-devel
cd csft-3.2.14
sh buildconf.sh
./configure --prefix=/usr/local/coreseek  --without-unixodbc \
--with-mmseg --with-mmseg-includes=/usr/local/mmseg3/include/mmseg/ \
--with-mmseg-libs=/usr/local/mmseg3/lib/ --with-mysql \
--with-mysql-includes=/usr/local/mysql/include \
--with-mysql-libs=/usr/local/mysql/lib
make && make install
```
中间很有可能会出错，如果出错按如下方法处理
编辑：
打开./src/MakeFile文件
将
LIBS = -lm -lexpat -L/usr/local/lib
改成
LIBS = -lm -lexpat -liconv -L/usr/local/lib

### 4. 安装测试
```
cd testpack  
cat var/test/test.xml    #此时应该正确显示中文  
/usr/local/mmseg3/bin/mmseg -d /usr/local/mmseg3/etc var/test/test.xml  
/usr/local/coreseek/bin/indexer -c etc/csft.conf --all  
/usr/local/coreseek/bin/search -c etc/csft.conf 网络搜索
```
如果正常会显示如下
```
words:    
1. '网络': 1 documents, 1 hits    
2. '搜索': 2 documents, 5 hits
```