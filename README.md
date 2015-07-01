## MAC 10.10.3 下通过brew安装PHP环境
  Brew 是 Mac 下面的包管理工具，通过 Github 托管适合 Mac 的编译配置以及 Patch，可以方便的安装开发工具

### 安装brew
	curl -LsSf http://github.com/mxcl/homebrew/tarball/master | sudo tar xvz -C/usr/local --strip 1  
	
### 查看brew是否安装成功
	brew -v

### 添加brew的PHP扩展库
	sudo brew update
	brew tap homebrew/dupes
	brew tap homebrew/php
	brew tap josegonzalez/homebrew-php

### 使用brew options php55命令查看安装时可以有哪些选项
	$ brew options php55
	--with-cgi
		Enable building of the CGI executable (implies --without-fpm)
	--with-debug
		Compile with debugging symbols
	--with-enchant
		Build with enchant support
	--with-gmp
		Build with gmp support
	--with-homebrew-apxs
		Build against apxs in Homebrew prefix
	--with-homebrew-curl
		Include Curl support via Homebrew
	--with-homebrew-libxml2
		Include Libxml2 support via Homebrew
	--with-homebrew-libxslt
		Include LibXSLT support via Homebrew
	--with-imap
		Include IMAP extension
	--with-libmysql
		Include (old-style) libmysql support instead of mysqlnd
	--with-mssql
		Include MSSQL-DB support
	--with-pdo-oci
		Include Oracle databases (requries ORACLE_HOME be set)
	--with-phpdbg
		Enable building of the phpdbg SAPI executable (PHP 5.4 and above)
	--with-postgresql
		Build with postgresql support
	--with-snmp
		Build with SNMP support
	--with-thread-safety
		Build with thread safety
	--with-tidy
		Include Tidy support
	--without-apache
		Disable building of shared Apache 2.0 Handler module
	--without-bz2
		Build without bz2 support
	--without-fpm
		Disable building of the fpm SAPI executable
	--without-ldap
		Build without LDAP support
	--without-mysql
		Remove MySQL/MariaDB support
	--without-pcntl
		Build without Process Control support
	--without-pear
		Build without PEAR
	--HEAD
		Install HEAD version

### 安装php55
	sudo brew install homebrew/php/php55

### 安装php55扩展
	sudo brew install php55-mcrypt        # mcrypt扩展
	sudo brew install php55-redis		  # redis扩展	
	sudo brew install php55-mysqlnd_ms    # mysql扩展

### 如果安装扩展时出现下面的错误提示
	错误提示:
		Error: Formulae found in multiple taps:
 			* homebrew/php/php55-mcrypt
 			* josegonzalez/php/php55-mcrypt
 	
		Please use the fully-qualified name e.g. homebrew/php/php55-mcrypt to refer the formula.

	解决方法: 
		sudo brew untap josegonzalez/homebrew-php

### 配置php.ini
	sudo vim /usr/local/etc/php/5.5/php.ini

	添加下面三行代码: 
		extension=redis.so
		extension=mcrypt.so
		extension=mysql.so

### 安装MySQL
	sudo brew install mysql

### 启动MySQL
	sudo mysql.server start
	如果提示下面错误:
		Starting MySQL
   		. ERROR! The server quit without updating PID file (/usr/local/var/mysql/*.local.pid).
	解决方法
		sudo chown -R _mysql:_mysql /usr/local/var/mysql

### 安装Redis
	sudo brew install redis

### 配置Redis并后台运行
	配置redis:
		sudo vim /usr/local/etc/redis.conf
			daemonize yes            # yes 以后台程序运行
	启动redis:
		redis-server /usr/local/etc/redis.conf		
