#### [jdk](http://tecadmin.net/install-oracle-java-8-jdk-8-ubuntu-via-ppa/)

```
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
```

#### [rvm](https://rvm.io/rvm/install)

```
curl -sSL https://get.rvm.io | bash -s stable --ruby --rails
```

#### [nodejs](https://github.com/nodesource/distributions)

```
curl -sL https://deb.nodesource.com/setup_4.x | sudo -E bash -
sudo apt-get install -y nodejs
```

#### [mysql-server](https://help.ubuntu.com/lts/serverguide/mysql.html)

```
sudo apt-get install mysql-server
```

#### [redis-server](http://blog.fens.me/linux-redis-install/)

```
sudo apt-get install redis-server
```

#### Upgrading Ruby With RVM

```
rvm upgrade 2.0.0
rvm use ruby-2.0.0 --default
```

#### [glances 进程管理](http://askubuntu.com/questions/293426/system-monitoring-tools-for-ubuntu)

```
sudo apt-get install python-pip build-essential python-dev lm-sensors
sudo pip install psutil logutils bottle batinfo https://bitbucket.org/gleb_zhulik/py3sensors/get/tip.tar.gz zeroconf netifaces pymdstat influxdb elasticsearch potsdb statsd pystache docker-py pysnmp pika py-cpuinfo bernhard
sudo pip install glances

glances
```
