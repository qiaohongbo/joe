# GitLab 源码安装

## 概述

gitlab 安装包括安装下列组件:

* 依赖包安装
* 安装 Ruby
* 安装 Go
* 用户设置
* 数据库安装配置
* 安装 Redis
* 安装 GitLab
* 安装配置 Nginx

### 安装依赖包

更新系统 ( 非必需 )

```
apt-get update -y
apt-get upgrade -y
```

安装必需的包

```
sudo apt-get install -y build-essential zlib1g-dev libyaml-dev libssl-dev libgdbm-dev libreadline-dev libncurses5-dev libffi-dev curl openssh-server checkinstall libxml2-dev libxslt-dev libcurl4-openssl-dev libicu-dev logrotate python-docutils pkg-config cmake nodejs
```
安装 git

```
sudo apt-get install -y git-core
＃ git 版本最好 > 1.9.0
git --version
```

### 安装 Ruby

先卸载系统自带 ruby

```
sudo apt-get remove ruby1.8
```

下载并编译

```
mkdir /tmp/ruby && cd /tmp/ruby
curl -O --progress https://cache.ruby-lang.org/pub/ruby/2.1/ruby-2.1.7.tar.gz
echo 'e2e195a4a58133e3ad33b955c829bb536fa3c075  ruby-2.1.7.tar.gz' | shasum -c - && tar xzf ruby-2.1.7.tar.gz
cd ruby-2.1.7
./configure --disable-install-rdoc
make
sudo make install
```

安装 bundler

```
sudo gem install bundler --no-ri --no-rdoc
```

### 安装 Go

```
curl -O --progress https://storage.googleapis.com/golang/go1.5.3.linux-amd64.tar.gz
echo '43afe0c5017e502630b1aea4d44b8a7f059bf60d7f29dfd58db454d4e4e0ae53  go1.5.3.linux-amd64.tar.gz' | shasum -c - && \
  sudo tar -C /usr/local -xzf go1.5.3.linux-amd64.tar.gz
sudo ln -sf /usr/local/go/bin/{go,godoc,gofmt} /usr/local/bin/
rm go1.5.3.linux-amd64.tar.gz
```

### 用户设置

```
sudo adduser --disabled-login --gecos 'GitLab' git
```

### 数据库安装配置

#### MySQL  这里使用源码编译也行

```
sudo apt-get install -y mysql-server mysql-client libmysqlclient-dev
# 5.5.14 或 最新版
mysql --version
# 设置 mysql 密码
sudo mysql_secure_installation

mysql -u root -p
# 创建用户
mysql> CREATE USER 'git'@'localhost' IDENTIFIED BY '$password';
# 设置引擎
mysql> SET storage_engine=INNODB;
# 创建数据库
mysql> CREATE DATABASE IF NOT EXISTS `gitlabhq_production` DEFAULT CHARACTER SET `utf8` COLLATE `utf8_unicode_ci`;
# 分配权限
mysql> GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, CREATE TEMPORARY TABLES, DROP, INDEX, ALTER, LOCK TABLES ON `gitlabhq_production`.* TO 'git'@'localhost';
# 远程登录测试一下
sudo -u git -H mysql -u git -p -D gitlabhq_production
```

### 安装 Redis

```
wget http://download.redis.io/releases/redis-2.8.23.tar.gz
tar xzf redis-2.8.23.tar.gz
cd redis-2.8.23
make

cd utils
sudo ./install_server.sh

sudo cp /etc/redis/redis.conf /etc/redis/redis.conf.orig

sed 's/^port .*/port 0/' /etc/redis/redis.conf.orig | sudo tee /etc/redis/redis.conf

echo 'unixsocket /var/run/redis/redis.sock' | sudo tee -a /etc/redis/redis.conf

echo 'unixsocketperm 770' | sudo tee -a /etc/redis/redis.conf

mkdir /var/run/redis
chown redis:redis /var/run/redis
chmod 755 /var/run/redis

if [ -d /etc/tmpfiles.d ]; then
  echo 'd  /var/run/redis  0755  redis  redis  10d  -' | sudo tee -a /etc/tmpfiles.d/redis.conf
fi

sudo service redis_6379 start

sudo usermod -aG redis git
```

### 安装 GitLab

```
# 进入 git 用户家目录
cd /home/git
```
克隆源码

```
sudo -u git -H git clone https://gitlab.com/gitlab-org/gitlab-ce.git -b 8-4-stable gitlab
```

配置

```
cd /home/git/gitlab

sudo -u git -H cp config/gitlab.yml.example config/gitlab.yml

# 配置 域名 第三方登录 邮箱 等...
sudo -u git -H vim config/gitlab.yml

sudo -u git -H cp config/secrets.yml.example config/secrets.yml
sudo -u git -H chmod 0600 config/secrets.yml

sudo chown -R git log/
sudo chown -R git tmp/
sudo chmod -R u+rwX,go-w log/
sudo chmod -R u+rwX tmp/

sudo chmod -R u+rwX tmp/pids/
sudo chmod -R u+rwX tmp/sockets/

# 这里如果提示目录不存在 则创建目录
sudo chmod -R u+rwX  public/uploads

sudo chmod -R u+rwX builds/

sudo chmod -R u+rwX shared/artifacts/

sudo -u git -H cp config/unicorn.rb.example config/unicorn.rb

sudo -u git -H cp config/initializers/rack_attack.rb.example config/initializers/rack_attack.rb

sudo -u git -H git config --global core.autocrlf input

sudo -u git -H cp config/resque.yml.example config/resque.yml

# 配置一下 redis
sudo -u git -H vim config/resque.yml
```

配置数据库

```
sudo -u git cp config/database.yml.mysql config/database.yml

# 配置数据库
sudo -u git -H vim config/database.yml

sudo -u git -H chmod o-rwx config/database.yml
```

安装 Ruby gem 包

```
sudo -u git -H bundle install --deployment --without development test postgres aws kerberos
```

安装 gitlab-shell

```
sudo -u git -H bundle exec rake gitlab:shell:install REDIS_URL=unix:/var/run/redis/redis.sock RAILS_ENV=production

sudo -u git -H vim /home/git/gitlab-shell/config.yml
```

安装 gitlab-workhorse

```
cd /home/git
sudo -u git -H git clone https://gitlab.com/gitlab-org/gitlab-workhorse.git
cd gitlab-workhorse
sudo -u git -H git checkout 0.5.4
sudo -u git -H make
```

初始化数据库并激活配置

```
cd /home/git/gitlab

sudo -u git -H bundle exec rake gitlab:setup RAILS_ENV=production

# 可以使用下面命令指定root密码
sudo -u git -H bundle exec rake gitlab:setup RAILS_ENV=production GITLAB_ROOT_PASSWORD=yourpassword
```

安装初始化脚步

```
sudo cp lib/support/init.d/gitlab /etc/init.d/gitlab

sudo cp lib/support/init.d/gitlab.default.example /etc/default/gitlab

sudo update-rc.d gitlab defaults 21
```

安装 Logrotate

```
sudo cp lib/support/logrotate/gitlab /etc/logrotate.d/gitlab
```

检测应用状态

```
sudo -u git -H bundle exec rake gitlab:env:info RAILS_ENV=production
```

编译静态资源

```
sudo -u git -H bundle exec rake assets:precompile RAILS_ENV=production
```

启动 gitlab

```
sudo service gitlab start
# or
sudo /etc/init.d/gitlab restart
```

### 安装配置 Nginx

安装 nginx

```
sudo apt-get install -y nginx
```

站点配置

```
sudo cp lib/support/nginx/gitlab /etc/nginx/sites-available/gitlab
sudo ln -s /etc/nginx/sites-available/gitlab /etc/nginx/sites-enabled/gitlab
```

配置站点域名

```
# Change YOUR_SERVER_FQDN to the fully-qualified
# domain name of your host serving GitLab.
# If using Ubuntu default nginx install:
# either remove the default_server from the listen line
# or else sudo rm -f /etc/nginx/sites-enabled/default

sudo vim /etc/nginx/sites-available/gitlab
```

测试 nginx

```
sudo nginx -t
```

启动 nginx

```
sudo service nginx restart
```


