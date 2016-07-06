
这是一个部署脚本

```shell
# 导出环境变量
export PATH=$PATH:%JAVA/bin
export CLASSPATH=$CLASSPATH:.:$JAVA_HOME/lib
export M2_HOME=/home/example/tool/maven
export PATH=$PATH:$M2_HOME/bin

# 创建一个临时文件夹，用于检出代码库common
mkdir -p /tmp/release/trunk/java
cd /tmp/release/trunk/java
rm -rf common 
svn co http://svn.example.com/svn/repo/trunk/java/common

# 创建一个临时文件夹，用于检出代码库mobile_api
mkdir -p /tmp/release/branches/java
cd /tmp/release/branches/java
rm -rf mobile_api
svn co http://svn.example.com/svn/repo/branches/java/mobile_api_for_dev_1.0.6

# 用maven进行构建
cd mobile_api_for_dev_1.0.6
mvn install

# 判断构建结果
if [ $? -ne 0 ]; then
	 echo 'pack failed';
     exit 1;
fi

# 替换之前的软件包
rm -rf /home/example/htdocs/mobile_api.war	  
cp target/mobile_api.war /home/example/htdocs/

cd /home/example/htdocs/

# 解压软件包
unzip -x mobile_api.war -d mobile_api_for_dev_tmp;
\cp -R -f web_test_properties/*.properties mobile_api_for_dev_tmp/WEB-INF/classes;
tar cvf /tmp/mobile_api_for_dev.tar mobile_api_for_dev/;
rm -rf mobile_api_for_dev/;
mv mobile_api_for_dev_tmp mobile_api_for_dev;

# kill掉tomcat进程，然后重启
ps -ef | grep mobile_api_for_dev | grep -v grep | awk '{print $2}' | xargs kill -9; 
sh /opt/tomcat_mobile_api_for_dev/bin/startup.sh;

```
