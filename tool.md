# RVM和Gemset

## 什么是Gemset
一个隔离的Ruby依赖项环境，避免交叉依赖项问题

## RVM
Ruby版本管理器

## RVM + Gemset
特定版本+以来隔离

## JRuby
纯Java实现的Ruby解释器。启动一个jvm，然后读取ruby脚步并解释执行，jprofiler可以监控Jruby

Java 执行jruby
```
java -jar $HOME/.rvm/rubies/jruby-9.2.14.0/lib/jruby.jar hello.rb

```

## 安装一个新版本
 ```
rvm install jruby-9.2.14.0
```

## 新安装一个Ruby版本会创建default和global两个版本
```
rvm gemset list
```

## 切换版本(可查看gemset路径)
```
rvm use 2.5.7
rvm use jruby-9.2.14.0@cytobank
```

## 查看gemset依赖
```
gem list
```

# 例子 foobar项目
## 安装Ruby 2.5.7
```
rvm install 2.5.7
rvm use 2.5.7
```

## 创建gemset
```
rvm gemset create foo
rvm gemset create bar
rvm use 2.5.7@foo
gem list
```

## 安装rails
```
gem install rails
gem list

rvm use 2.5.7@bar
gem list
```

foo有rails bar没有

# rail 命令
```
bin/rails server
bin/rails console

bin/rails generate scaffold article
bin/rails generate model book name:string{20} description:text

bin/rails routes

bin/rails db:create
bin/rails db:migrate
```
