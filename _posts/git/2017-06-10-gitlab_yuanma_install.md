---
layout: blog
istop: true
title: "gitlab 源码安装"
background-image: 
date:  2017-06-10
category: git
tags:
- github
- Git源码安装
---
[TOC]
0.开篇的话
文章依赖 1. 官方博客 2. 配置网络yum仓库 提取密码ak1v 3. centos下安装gitlab 4. centos7下源码安装gitlab
出现错误解决方案依赖
1. 提示没有找不到 Specified 'mysql2' 2. bundle更换源问题 3. no tmp uploads folder yet
1.关闭SELINUX和iptables
[root@localhost ~]# service iptables stop       #关闭防火墙
[root@localhost ~]# setenforce 0                #暂时关闭Selinux

2.添加EPEL存储库
    wget -O /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6 https://getfedora.org/static/0608B895.txt
    rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-6
    
    #验证密钥安装成功
    rpm -qa gpg*    
    
    #安装软件包，不区分32和64位
    rpm -Uvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
3.添加Remi的RPM存储库
    wget -O /etc/pki/rpm-gpg/RPM-GPG-KEY-remi http://rpms.famillecollet.com/RPM-GPG-KEY-remi
    rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-remi
    
    ##验证密钥安装成功
    rpm -qa gpg*
    
    #安装软件包
    rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
    
    #查看上述仓库是否启动
    yum repolist
    
    repo id        repo name                                                       status
    base           CentOS-6 - Base                                                  6696
    epel           Extra Packages for Enterprise Linux 6 - x86_64                  12125
    extras         CentOS-6 - Extras                                                  61
    remi-safe      Safe Remi's RPM repository for Enterprise Linux 6 - x86_64        827
    updates        CentOS-6 - Updates                                                137
    repolist: 19846
    
4.安装GitLab所需的工具
    yum -y update
    
    #安装需要的开发的工具
    yum -y groupinstall 'Development Tools'                                     
    
    yum -y install readline readline-devel ncurses-devel gdbm-devel glibc-devel tcl-devel openssl-devel curl-devel expat-devel db4-devel byacc sqlite-devel libyaml libyaml-devel libffi libffi-devel libxml2 libxml2-devel libxslt libxslt-devel libicu libicu-devel system-config-firewall-tui redis sudo wget crontabs logwatch logrotate perl-Time-HiRes git cmake libcom_err-devel.i686 libcom_err-devel.x86_64 nodejs
    
    yum -y install python-docutils
5.安装邮件服务器
    #在这里自行可以自行去配置，官方推荐的是postfix。有机会的话，笔者会研究一下，后续更新上来博客
6.源码安装git
    #必须要确定git的版本高于2.7.4或更高版本。如果系统安装了git那么需要卸载
    yum -y remove git
    
    #安装git编译需求的软件
    yum install zlib-devel perl-CPAN gettext curl-devel expat-devel gettext-devel openssl-devel
    
    #下载源码并安装
    mkdir /tmp/git && cd /tmp/git
    curl --progress https://www.kernel.org/pub/software/scm/git/git-2.9.0.tar.gz | tar xz
    cd git-2.9.0
    ./configure
    make
    make prefix=/usr/local install
7.安装Ruby
    #删除旧的Ruby 1.8软件包（如果存在）。GitLab只支持Ruby 2.1版本系列
    yum remove ruby
    
    #下载并编译
    mkdir /tmp/ruby && cd /tmp/ruby
    curl --progress https://cache.ruby-lang.org/pub/ruby/2.1/ruby-2.1.9.tar.gz | tar xz
    cd ruby-2.1.9
    ./configure --disable-install-rdoc
    make
    make prefix=/usr/local install
    
    #安装Bundler Gem
    gem install bundler --no-doc
    
    # 检测信息，如果输出信息就代表没错
    which ruby
    ruby -v
8.安装Go
    #从GitLab 8.0开始，Git HTTP请求由gitlab-workhorse（以前称为gitlab-git-http-server）处理。这是一个在Go写的小守护进程。要安装gitlab-workhorse，我们需要一个Go编译器。
    yum install -y golang golang-bin golang-src
8.创建用户
   #Gitlab 创建一个用户
   adduser --system --shell /bin/bash --comment 'GitLab' --create-home --home-dir /home/git/ git
   
   #修改sudoers文件
   visudo
   Defaults    secure_path = /sbin:/bin:/usr/sbin:/usr/bin:/usr/local/bin
9.安装数据库
    #官方推荐用PostgreSQL，这里我们用Mysql来安装
    #安装软件
    yum install -y mysql-server mysql-devel
    chkconfig mysqld on
    service mysqld start
    
    #这里Mysql的版本保持这个样子就可以，亲测可用
    mysql --version
    #mysql  Ver 14.14 Distrib 5.1.73, for redhat-linux-gnu (x86_64) using readline 5.1

    #首先对数据库进行一下设置
    mysql_secure_installation
    
    #登陆系统
    mysql -u root -p
    
    #为gitlab创建用户
    CREATE USER 'git'@'localhost' IDENTIFIED BY 'your passwd';
    
    #添加InnoDB引擎
    SET storage_engine=INNODB;
    
    #下面先不创建Gitlab生产数据库，后面有个命令可以创建这里先不创建(不执行)
    #CREATE DATABASE IF NOT EXISTS `gitlabhq_production` DEFAULT CHARACTER SET `utf8` COLLATE `utf8_unicode_ci`;
    
    #授权
    GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, CREATE TEMPORARY TABLES, DROP, INDEX, ALTER, LOCK TABLES, REFERENCES ON `gitlabhq_production`.* TO 'git'@'localhost';
    
    #退出数据库
    \q
10.安装Redis
    #Gitlab要求Redis版本不低于2.8，系统默认安装版本为2.4.1，卸载当前版本并采用remi源安装最新版本：
    #卸载当前redis：
    yum -y remove redis
    #采用remi源安装最新版本：
    yum --enablerepo=remi install redis

    #确保redis在启动时启动：
    chkconfig redis on
    
    #将redis配置为使用套接字：
    cp /etc/redis.conf /etc/redis.conf.orig
    
    #通过将'port'设置为0，禁止在TCP上侦听Redis：
    sed 's/^port .*/port 0/' /etc/redis.conf.orig | sudo tee /etc/redis.conf

    #为默认CentOS路径启用Redis套接字：
    echo 'unixsocket /var/run/redis/redis.sock' | sudo tee -a /etc/redis.conf
    echo -e 'unixsocketperm 0770' | sudo tee -a /etc/redis.conf

    #创建包含套接字的目录
    mkdir /var/run/redis
    chown redis:redis /var/run/redis
    chmod 755 /var/run/redis
    
    #保留包含套接字的目录（如果适用）
    if [ -d /etc/tmpfiles.d ]; then
        echo 'd  /var/run/redis  0755  redis  redis  10d  -' | sudo tee -a /etc/tmpfiles.d/redis.conf
    fi
    
    #修改配置文件
    vim /etc/redis.conf
        #bind 127.0.0.1
 
    #重启服务：
    service redis restart
    
    #如果遇到以下问题
    
    #>>> 'vm-enabled no'
    #Bad directive or wrong number of arguments
                                                           [FAILED]

    只需要注释以下内容：
    #vm-enabled yes
    #vm-swap-file /tmp/redis.swap
    #vm-max-memory 0
    #vm-page-size 32
    #vm-pages 134217728
    #vm-max-threads 4
    #hash-max-zipmap-entries 512
    #hash-max-zipmap-value 64
    
    #将git添加到redis组：
    usermod -aG redis git
11.安装Gitlab
    #存放gitlab等相关软件集合
    cd /home/git
    
    #下载
    sudo -u git -H git clone https://gitlab.com/gitlab-org/gitlab-ce.git -b 8-9-stable gitlab
    
    #配置，从这里开始直接复制即可，主要就是创建目录，赋予权限
    
    # Go to GitLab installation folder
    cd /home/git/gitlab
    
    # Copy the example GitLab config
    sudo -u git -H cp config/gitlab.yml.example config/gitlab.yml
    
    # Update GitLab config file, follow the directions at top of file
    sudo -u git -H editor config/gitlab.yml
    
    # Copy the example secrets file
    sudo -u git -H cp config/secrets.yml.example config/secrets.yml
    sudo -u git -H chmod 0600 config/secrets.yml
    
    # Make sure GitLab can write to the log/ and tmp/ directories
    sudo chown -R git log/
    sudo chown -R git tmp/
    sudo chmod -R u+rwX,go-w log/
    sudo chmod -R u+rwX tmp/
    
    # Make sure GitLab can write to the tmp/pids/ and tmp/sockets/ directories
    sudo chmod -R u+rwX tmp/pids/
    sudo chmod -R u+rwX tmp/sockets/
    
    # Create the public/uploads/ directory
    sudo -u git -H mkdir public/uploads/
    
    # Make sure only the GitLab user has access to the public/uploads/ directory
    # now that files in public/uploads are served by gitlab-workhorse
    sudo chmod 0700 public/uploads
    
    sudo chmod ug+rwX,o-rwx /home/git/repositories/
    
    # Change the permissions of the directory where CI build traces are stored
    sudo chmod -R u+rwX builds/
    
    # Change the permissions of the directory where CI artifacts are stored
    sudo chmod -R u+rwX shared/artifacts/
    
    # Copy the example Unicorn config
    sudo -u git -H cp config/unicorn.rb.example config/unicorn.rb
    
    # 查看核心数
    nproc
    
    # Enable cluster mode if you expect to have a high load instance
    # Ex. change amount of workers to 3 for 2GB RAM server
    # Set the number of workers to at least the number of cores
    sudo -u git -H editor config/unicorn.rb
    
    # Copy the example Rack attack config
    sudo -u git -H cp config/initializers/rack_attack.rb.example config/initializers/rack_attack.rb
    
    # Configure Git global settings for git user
    # 'autocrlf' is needed for the web editor
    sudo -u git -H git config --global core.autocrlf input
    
    # Disable 'git gc --auto' because GitLab already runs 'git gc' when needed
    sudo -u git -H git config --global gc.auto 0
    
    # Configure Redis connection settings
    sudo -u git -H cp config/resque.yml.example config/resque.yml
    
    # 这里保持默认
    sudo -u git -H vim config/resque.yml
11.数据库配置
    #配置数据库文件
    sudo -u git cp config/database.yml.mysql config/database.yml
    
    #编辑数据库文件
    sudo -u git -H vim config/database.yml
      production:
      adapter: mysql2
      encoding: utf8
      collation: utf8_general_ci
      reconnect: false
      database: gitlabhq_production
      pool: 10
      username: gitlab
      password: "gitlab"
      # host: localhost
      # socket: /tmp/mysql.sock
      
    #赋予权限
    sudo -u git -H chmod o-rwx config/database.yml
12.安装Gems
    cd /home/git/gitlab

   
    sudo -u git -H bundle config build.pg --with-pg-config=/usr/pgsql-9.3/bin/pg_config
    sudo -u git -H bundle install --deployment --without development test mysql aws kerberos


    sudo -u git -H bundle install --deployment --without development test postgres aws kerberos
    #提示出现这个错误：
    #Gem::Ext::BuildError: ERROR: Failed to build gem native extension.
    #/usr/local/bin/ruby extconf.rb --with-pg-config=/usr/pgsql-9.3/bin/pg_config
    #Using config values from /usr/pgsql-9.3/bin/pg_config
    #sh: /usr/pgsql-9.3/bin/pg_config: No such file or directory
    #sh: /usr/pgsql-9.3/bin/pg_config: No such file or directory
    
    #解决方法：
        
        #卸载
        yum remove postgresql
        
        #安装pgdg仓库：
        rpm -Uvh http://yum.postgresql.org/9.3/redhat/rhel-6-x86_64/pgdg-centos93-9.3-2.noarch.rpm
        
        #安装postgresql93-server，postgreqsql93-devel和postgresql93-contrib库：
        yum install postgresql93-server postgresql93-devel postgresql93-contrib
        
        #重命名服务脚本：
        mv /etc/init.d/{postgresql-9.3,postgresql}
        
        # 服务启动
        service postgresql initdb
        service postgresql start
        chkconfig postgresql on
    
    #这个时候就不会报错了，提示以下信息
    
        #-------------------------------------------------
        #Thank you for installing html-pipeline!
        #You must bundle Filter gem dependencies.
        #See html-pipeline README.md for more details.
        #https://github.com/jch/html-pipeline#dependencies
        #-------------------------------------------------
    
    #这里我们修改一个Gemfile文件，刚刚编辑的数据库配置文件我们看到了Mysql2，我们执行的上述步骤是没有安装的，检查的方法可以使用如下
        
        #查看mysql2是否安装
        bundle show mysql2
        
        #当前目录在Gitlab目录下，编辑Gemfile文件，修改成如下内容
        vim Gemfile
            gem "mysql2", '~> 0.3.18'
        
        #在执行以下语句
        bundle install --no-deployment
        
        #执行时间根据网速和电脑配置而言，下面查看以下结果
        [root@localhost gitlab]# bundle show mysql2
        /usr/local/lib/ruby/gems/2.1.0/gems/mysql2-0.3.20

13.安装GitLab Shell
    
     #安装  
    sudo -u git -H bundle exec rake gitlab:shell:install[v3.0.0] REDIS_URL=unix:/var/run/redis/redis.sock RAILS_ENV=production

    #编辑文件  这个文件主要就是提供gitlab API 接口的
    sudo -u git -H vim /home/git/gitlab-shell/config.yml
        #修改内容如下
        gitlab_url: "http://localhost:8000/"

    #这个文件主要提供ruby的服务端口和ip
    [root@localhost gitlab]# vim config/unicorn.rb
        #修改内容如下
        listen "127.0.0.1:8080", :tcp_nopush => true
    
    #这个文件主要提供gitlab服务的端口，ip
    [root@localhost gitlab]# vim config/gitlab.yml
        #修改内容如下
        gitlab:
            host: localhost
            port: 8000 
        git:
            bin_path: /usr/local/bin/git
    
    #如果后面的操作会提示git找不到那么需要执行以下操作
    ln -s /usr/local/bin/git /usr/bin/git
    
    restorecon -Rv /home/git/.ssh
14.安装gitlab-workhorse
    #这个不多说了，很简单
    cd /home/git
    sudo -u git -H git clone https://gitlab.com/gitlab-org/gitlab-workhorse.git
    cd gitlab-workhorse
    sudo -u git -H git checkout v0.7.5
    sudo -u git -H make
15.初始化数据库
    cd /home/git/gitlab
    sudo -u git -H bundle exec rake gitlab:setup RAILS_ENV=production
    
    #这里会提示如下“错误”
    #Couldn't drop gitlabhq_production
    #Access denied for user 'gitlab'@'localhost' (using password: YES)Please provide the root password for your MySQL installation
    >输入你设定的mysql密码
    
    #之后就会提示如下内容
    #   -> 0.0104s
    #-- add_foreign_key("u2f_registrations", "users")
    #   -> 0.0085s
    #-- initialize_schema_migrations_table()
    #   -> 0.0188s
    #Adding limits to schema.rb for mysql
    #-- change_column(:merge_request_diffs, :st_commits, :text, {:limit=>2147483647})
    #   -> 0.0218s
    #-- change_column(:merge_request_diffs, :st_diffs, :text, {:limit=>2147483647})
    #   -> 0.0107s
    #-- change_column(:snippets, :content, :text, {:limit=>2147483647})
    #   -> 0.0134s
    #-- change_column(:notes, :st_diff, :text, {:limit=>2147483647})
    #   -> 0.0149s
    #-- change_column(:events, :data, :text, {:limit=>2147483647})
    #   -> 0.0096s

    #== Seed from /home/git/gitlab/db/fixtures/production/001_admin.rb
    #Administrator account created:
    
    #login:    root
    #password: You'll be prompted to create one on your first visit.

    
    #当你进行登陆的时候就会提示你更改密码的操作
    

16.安装启动脚本
    #安装了这么长时间终于快结束了，心情有没有开心！！！！
    #当前目录在/home/git/gitlab
    cp lib/support/init.d/gitlab /etc/init.d/gitlab
    chkconfig gitlab on
    cp lib/support/logrotate/gitlab /etc/logrotate.d/gitlab
    
    #检查程序的版本
    #System information
    #System:		CentOS 6.8
    #Current User:	git
    #Using RVM:	no
    #Ruby Version:	2.1.9p490
    #Bundler Version:1.14.6
    #Rake Version:	10.5.0
    #Sidekiq Version:4.1.2
    
    #GitLab information
    #Version:	8.9.11
    #Revision:	9a05855
    #Directory:	/home/git/gitlab
    #DB Adapter:	mysql2
    #URL:		http://localhost:8000
    #HTTP Clone URL:	http://localhost:8000/some-group/some-project.git
    #SSH Clone URL:	git@localhost:some-group/some-project.git
    #Using LDAP:	no
    #Using Omniauth:	no
    
    #GitLab Shell
    #Version:	3.0.0
    #Repositories:	/home/git/repositories/
    #Hooks:		/home/git/gitlab-shell/hooks/
    #Git:		/usr/bin/git
   
    #生成网页需要的资源
    sudo -u git -H bundle exec rake assets:precompile RAILS_ENV=production
    
    
    #启动实例
    service gitlab start
    
    #如果看到以下信息，就代表你已经成功安装了Gitlab
    #Starting GitLab Unicorn
    #Starting GitLab Sidekiq
    #Starting Gi充电tLab Workhorse
    
    #The GitLab Unicorn web server with pid 80821 is running.
    #The GitLab Sidekiq job dispatcher with pid 80870 is running.
    #The GitLab Workhorse with pid 80852 is running.
    #GitLab and all its components are up and running.
    
    #下面在浏览器开始c访问
    http://localhost:8000
    #开始就会弹出让你修改密码的界面，之后就提示登陆界面
    #http://localhost:8000/users/password/edit?reset_password_token=DjzQ-AcPsgdkhx4ckUBg
    #初始的时候你可以用root和你刚刚设定的密码进行登陆
    #http://localhost:8000/users/sign_in
    
    #如果发现不能登陆，请确保服务启动成功，然后在重点检查以下配置文件
    #当前目录   /home/git/gitlab
    vim config/gitlab.yml
    vim config/unicorn.rb
    vim ../gitlab-shell/config.yml
    
    
    #如果提示以下信息，那么需要检测Redis,mysql是否启动
    /etc/init.d/gitlab restart
    The GitLab Unicorn web server is not running.
    #查看/home/git/gitlab/log/unicorn.stderr.log，最新的日志在末尾。
    /home/git/gitlab/config/environments/production.rb:82: syntax error
    #进入命令模式set nu显示行号，找到82行所在。
    
    #如果发现启动不了服务，可以尝试重新创建一下数据库
    sudo -u git -H bundle exec rake gitlab:setup RAILS_ENV=production
    
    #检测一下程序状态
    #这里可以看出你的版本是否达到要求，不过笔者一切比较正常。
    #前几次装的时候出现过版本不对，不过也没有影响使用
    #可能这是个bug吧 如果那位大神了解这个还请给我留言，谢谢
    sudo -u git -H bundle exec rake gitlab:check RAILS_ENV=production
    
        #Checking GitLab Shell ...
        #
        #GitLab Shell version >= 3.0.0 ? ... OK (3.0.0)
        #Repo base directory exists? ... yes
        #Repo base directory is a symlink? ... no
        #Repo base owned by git:git? ... yes
        #Repo base access is drwxrws---? ... yes
        #hooks directories in repos are links: ... can't check, you have no projects
        #Running /home/git/gitlab-shell/bin/check
        #Check GitLab API access: OK
        #Check directories and files: 
        #	/home/git/repositories/: OK
        #	/home/git/.ssh/authorized_keys: OK
        #Send ping to redis server: gitlab-shell self-check successful
        #
        #Checking GitLab Shell ... Finished
        #
        #Checking Sidekiq ...
        #
        #Running? ... yes
        #Number of Sidekiq processes ... 1
        #
        #Checking Sidekiq ... Finished
        #
        #Checking Reply by email ...
        #
        #Reply by email is disabled in config/gitlab.yml
        #
        #Checking Reply by email ... Finished
        #
        #Checking LDAP ...
        #
        #LDAP is disabled in config/gitlab.yml
        #
        #Checking LDAP ... Finished
        #
        #Checking GitLab ...
        #
        #Git configured with autocrlf=input? ... yes
        #Database config exists? ... yes
        #All migrations up? ... yes
        #Database contains orphaned GroupMembers? ... no
        #GitLab config exists? ... yes
        #GitLab config outdated? ... no
        #Log directory writable? ... yes
        #Tmp directory writable? ... yes
        #Uploads directory setup correctly? ... skipped (no tmp uploads folder yet)
        #Init script exists? ... yes
        #Init script up-to-date? ... yes
        #projects have namespace: ... can't check, you have no projects
        #Redis version >= 2.8.0? ... yes
        #Ruby version >= 2.1.0 ? ... yes (2.1.9)
        #Your git bin path is "/usr/local/bin/git"
        #Git version >= 2.7.3 ? ... yes (2.9.0)
        #Active users: 1
        #
        #Checking GitLab ... Finished

17. Nginx服务器安装
    #说在前头： 这里的web服务器可以选择性安装，就是可以通过外网正常访问你的GitLab,这里web服务器可以选择Nginx或这Apache
    
    sudo yum install -y nginx
    sudo cp lib/support/nginx/gitlab /etc/nginx/conf.d/gitlab.conf
    
    #这里默认不修改端口 默认是80
    vim /etc/nginx/conf.d/gitlab.conf 
    vim /etc/nginx/conf.d/default.conf 
    vim /etc/nginx/nginx.conf

    #测试语法
    sudo nginx -t
    
    #启动服务
    sudo service nginx restart
    chkconfig nginx on
    sudo chmod 775 /home/git
   
    #在本地电脑访问对于的IP。如果以上均配置正确，就会看到登陆页面
    http://192.168.223.140:80
    
