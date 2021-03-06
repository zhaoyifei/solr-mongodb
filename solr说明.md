# solr 操作

## 1 安装 jdk
```bash
wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u101-b13/jdk-8u101-linux-x64.rpm

http://download.oracle.com/otn-pub/java/jdk/8u101-b13/jdk-8u101-linux-x64.rpm
rpm -i jdk-8u101-linux-x64.rpm
java -version 
版本是sunjdk 1.8
```
## 2 下载安装solr

```bash
5.3
wget "http://apache.fayea.com/lucene/solr/5.3.1/solr-5.3.1.zip"
4.10
wget http://apache.fayea.com/lucene/solr/4.10.4/solr-4.10.4.zip
6.1
wget http://mirror.bit.edu.cn/apache/lucene/solr/6.1.0/solr-6.1.0.zip

## 3 启动solr

5.3版本
root/solr-5.3.1/bin/solr start -e cloud -noprompt -m 3g -a "-XX:MaxDirectMemorySize=2g"

6.1版本 上面的命令有问题
bin/solr -e cloud -noprompt

/停止solr/
/root/solr-5.3.1/bin/solr stop -all

## 4 索引

solr-5.3.1/bin/post -c gettingstarted docs/

## 5 创建core

/root/solr-5.3.1/bin/solr create_core -c doctor -d /root/solr-5.3.1/conf
/root/solr-5.3.1/bin/solr delete -c doctor

----------------------------------------------------------------------

# 分词 IK-Analyzer

cp IKAnalyzer2012FF_u2.jar ./solr-5.3.1/server/solr-webapp/webapp/libs/
mkdir ./solr-5.3.1/server/solr-webapp/webapp/WEB-INF/classes
cp IKAnalyzer.cfg.xml stopword.dic ./solr-5.3.1/server/solr-webapp/webapp/WEB-INF/classes

  
----------------------------------------------------------------------------
# 连接 mongodb

## 1 安装mongodb 
Replica Set

/root/mongodb/bin/mongod --dbpath /root/mongo/data/db1 --logpath /root/mongo/log1/mongo.log --port 27020 --replSet solrT --fork
/root/mongodb/bin/mongod --dbpath /root/mongo/data/db2 --logpath /root/mongo/log2/mongo.log --port 27021 --replSet solrT --fork
/root/mongodb/bin/mongod --dbpath /root/mongo/data/db3 --logpath /root/mongo/log3/mongo.log --port 27022 --replSet solrT --fork

config = {_id:"solrT",members:[
{_id:0,host:'127.0.0.1:27020'},
{_id:1,host:'127.0.0.1:27021'},
{_id:2,host:'127.0.0.1:27022'}]
 }
 
 rs.initiate(config)
 
 rs.status()

-------------------------------------------------------
## 2 CentOS6.4 安装 mongo-connector

mongo-connector在python2.6.6版本下安装不成功，官方测试2.7,3.3正常

### 需要升级python2.7

具体步骤：

#### 安装开发工具包：
yum groupinstall "Development tools"
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel
（编译安装python2.7.5，没有zlib-devel,可以编译成功，但是当程序有调用zlib的时候会报tarfile.ReadError: file could not be opened successfully）

#### 安装python，修改yum相关内容，文章中最后拷贝rpm包如果yum正常可以省略（我的正常）

参见http://www.linuxidc.com/Linux/2013-05/84727.htm

python就绪之后


 github 下载 源码

解压后，首先进入解压后目录安装 easyinstall : sudo python ez_setup.py

然后安装 mongo-connector: sudo python setup.py install 




#### 因开发需要，今天把CentOS 6.4自带的Python2.6.6升级到了Python2.7.3.按照如下步骤进行升级

1、查看当前系统python的版本

1
python -V
2、下载2.7.3版本的Python

1
wget http://python.org/ftp/python/2.7.3/Python-2.7.3.tar.bz2

https://www.python.org/ftp/python/3.5.0/Python-3.5.0.tgz
3、解压和安装


tar -jxvf Python-2.7.3.tar.bz2
#进入解压后的目录
cd Python-2.7.3
#编译和安装
./configure
make
make install
4.查看是否安装成功


/usr/local/bin/python2.7 -V
#如果出现如下信息代表安装成功
Python 2.7.3
5、建立软链接

#正常情况下即使python2.7安装成功后，系统默认指向的python仍然是2.6.6版本，考虑到yum是基于python2.6.6才能正常工作，所以不建议卸载。
#采用下面的方法把系统默认的python修改为2.7.3版本
mv /usr/bin/python /usr/bin/python2.6.6
ln -s /usr/local/bin/python2.7 /usr/bin/python
#检测是否成功
python -V
#出现2.7.3版本信息代表成功
Python 2.7.3
6、解决修改完系统默认python版本后yum不可用的问题

#修改yum文件
vi /usr/bin/yum
将文件头部的

1
#!/usr/bin/python
改为如下内容

1
#!/usr/bin/python2.6.6
整个升级过程完成了。

-------------------------------------------------
#### 安装pip
wget --no-check-certificate https://pypi.python.org/packages/source/p/pip/pip-7.1.2.tar.gz#md5=3823d2343d9f3aaab21cf9c917710196
tar zvxf pip-7.1.2.tar.gz
cd pip-7.1.2
python setup.py install
pip install pymongo

-------------------------------------------------
## mongo-connector
mongo 连接 solr
python

git clone https://github.com/10gen-labs/mongo-connector.git

cd mongo-connector
python setup.py install

### mongo-connector -m localhost:27217 -t http://localhost:8983/solr


### mongo-connector -m 127.0.0.1:27020 -t http://127.0.0.1:8983/solr/collection1 --batch-size=100 -o oplog_progress.txt -n zlyweb.profiles -u _id -d /root/mongo-connector/mongo_connector/doc_managers/solr_doc_manager.py

### mongo-connector --auto-commit-interval=1 -d solr_doc_manager -t http://localhost:8983/solr/techproducts

this command is ok
mongo-connector --auto-commit-interval=1   -m 127.0.0.1:27020  -o oplog_progress.txt -n zlyweb.profiles -u _id  --batch-size=100 -t http://192.168.1.35:8983/solr/core1 -d solr_doc_manager

mongo-connector --auto-commit-interval=1   -m 127.0.0.1:27020  -o oplog_progress.txt -n zlyweb.profiles -u _id  --batch-size=100 -t http://192.168.1.35:8983/solr/doctor3 -d solr_doc_manager





http://192.168.1.35:8983/solr/doctor46/select?q=infos%3A“心脏主治”&start=50&rows=10&wt=json&indent=true


/usr/local/bin/mongo-connector --auto-commit-interval=1   -m 10.165.64.109:27017  -o oplog_progress.txt -n zlyweb.profiles -u _id  --batch-size=100 -t http://10.51.176.134:8983/solr/doctor9 -d solr_doc_manager
/home/zlycare/solr-5.3.1/bin/solr
