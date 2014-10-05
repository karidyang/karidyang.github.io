---
layout: post
title: "部署rails生产环境"
date: 2014-08-04 21:12:01 +0800
comments: true
categories: rails
---





#####1、添加Gem

```
gem 'unicorn'
gem 'execjs'
```

#####2、添加unicorn的配置文件

```
touch config/unicorn.conf
vi config/unicorn.conf
rails_env = ENV['RAILS_ENV'] || 'production'
user 'root'
worker_processes 2
preload_app true
timeout 30
listen 3009
working_directory "/www/htdocs/weixin_backend"
pid "/www/htdocs/weixin_backend/unicorn.pid"
stderr_path "/www/htdocs/weixin_backend/log/unicorn.stderr.log"
stdout_path "/www/htdocs/weixin_backend/log/unicorn.stdout.log"
```

#####3、添加启动脚本

```
unicorn_rails -E production -c config/unicorn.rb -D
```

#####4、启动之前需要先执行

```
rake assets/precompile
```

#####5、配置nginx

```
location /assets 
{
	root /www/htdocs/weixin_backend/public;
}
```