# ELK部署

### 所用服务器：

测试应用服务器A		192.168.104.48 root/root	


测试应用服务器B		192.168.104.49 root/root	

测试应用服务器C		192.168.104.50 root/root

测试应用服务器D		192.168.104.51 root/root	


测试应用服务器E		192.168.104.52 root/root	部署logStash 端口9600

测试应用服务器F		192.168.104.53 root/root   部署kibana 端口5601

测试应用服务器G		192.168.104.55 root/root   部署ElasticSearch 端口9200   部署logStash 端口9600

测试应用服务器H		192.168.104.56 root/root

### 部署步骤

#### 1、下载：

分别下载ELK，

E地址：https://www.elastic.co/cn/downloads/elasticsearch

L地址：https://www.elastic.co/cn/downloads/logstash

K地址：https://www.elastic.co/cn/downloads/kibana

#### 2、安装：

##### 2.1安装E

创建elasticSearch用户：useradd elkuser

检查java版本(要求1.8以上)：java -version

解压下载好的ElasticSearch：tar -xzvf elasticsearch-7.0.0-rc1-linux-x86_64.tar.gz 

在文件目录下建文件夹data和logs（如果不存在的话）：cd elasticsearch-7.0.0-rc1/

​							   					  mkdir data

修改config目录下的配置文件：cd config

​						     vim elasticsearch.yml

修改内容：node.name: node-1		==》node.name: node-55

​		   path.data: /path/to/data		==》path.data: /home/elk/elasticsearch-7.0.0-rc1/data

​		   path.logs: /path/to/logs 		==》path.logs: /home/elk/elasticsearch-7.0.0-rc1/ogs

​		  bootstrap.memory_lock: true 		==》bootstrap.memory_lock: false

​		  network.host: 192.168.0.1 		==> network.host: 192.168.104.55

修改运行线程数后重启： echo "* soft nofile 65536" >> /etc/security/limits.conf

​					echo "* hard nofile 131072" >> /etc/security/limits.conf 

​					echo "elkuser soft nproc 4096" >> /etc/security/limits.conf

​					echo "elkuser hard nproc 4096" >> /etc/security/limits.conf 

​					reboot

如果使用root解压的ElasticSearch，需要执行：chown -R elkuser:elkuser elasticsearch-7.0.0-rc1

切换ElasticSearch用户，启动服务：bin/elasticsearch

报错：1、 max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
	    2、 the default discovery settings are unsuitable for production use; at least one of 							    [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured

处理[1]:执行（重启后失效）:sysctl -w vm.max_map_count=262144

​	     查看是否修改成功：sysctl -a|grep vm.max_map_count

​	     永久有效的方法：在文件/etc/sysctl.conf中加入一行：vm.max_map_count=262144

处理[2]:修改配置文件 vim config/elasticsearch.yml

​		cluster.initial_master_nodes: ["node-1", "node-2"]    ==》cluster.initial_master_nodes: ["node-55"]

​		在文件后增加两行

​		http.cors.enabled: true
​		http.cors.allow-origin: "*"：

目前为止部署成功！

##### 2.2安装K

解压：tar -zxvf kibana-7.0.0-rc1-linux-x86_64.tar.gz 

修改配置文件：vim config/kibana.yml
			server.host:"192.168.104.53"

​			server.name: "your-hostname" ==》server.name: "kibana53"

​			elasticsearch.hosts: ["http://localhost:9200"]  ==>  elasticsearch.hosts: ["http://192.168.104.55:9200"]

​			server.host: "localhost"  ==》server.host: "192.168.104.53"

启动：执行： bin/kibana

​	   如果多次启动导致端口被占用，先执行：lsof -i :5601 再找到对应的pid kill掉

目前为止部署成功！

##### 2.3安装L

解压：tar -zxvf logstash-7.0.0-rc1.tar.gz 

创建配置文件：vim logstash-devicemonitor2-test-log.conf

配置文件的内容：

```
input {
    file {
        path => "/home/smarthome/logs/superManagement.log"
        start_position => beginning
    }
}
filter {
    
}
output {
	elasticsearch{
	hosts => "192.168.104.55:9200"
	index => "logstash-devicemonitor2-test-log"
	}
	stdout {codec => rubydebug}
}
```
启动配置文件(后台启动)：nohup bin/logstash -f logstash-devicemonitor2-test-log.conf  

#### 3、调用链：

日志系统-->***.log-->logStash-->index名称-->E-->K



​			









  











