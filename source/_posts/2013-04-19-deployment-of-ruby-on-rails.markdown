---
layout: post
title: "Deployment of Ruby On Rails"
date: 2013-04-19 01:17
comments: true
categories: [Ubuntu 12.04, Ruby, RubyOnRails, MySQL, Nginx ]  
---

From time to time I need to refresh a guide of Rails application production environment deployment.

My favourite OS is Linux Ubuntu. I prefer to use LTS versions, and today's LTS is 12.04 Precise Pangolin. You can download server distribution from [Ubuntu official site](http://ubuntu.com).

I will not cover OS installation process, since thats quite trivial and there are a lot of tutorials in the Internet about that. Besides, I can imagine that OS already installed on your VPS/PS/ whatever you have.

*Note:* You might want to use chef or puppets for deployment automatization. And you are absolutelly right - recipes are the best way to DRY that as well. I will definitely go for it at some point. 

So you have installed OS and logged in to the server via ssh. 

*Note:* Don't forget to disable passworded authorization and root login for your ssh server. Also its nice to change ssh port from default one to some thing else, so the hackers will not be able to use simples atacks. 

    sudo sed -i "/^Port / c Port 22022" /etc/ssh/sshd_config
    sudo sed -i "/^PermitRootLogin / c PermitRootLogin no" /etc/ssh/sshd_config

Here is a set of commands you need to execute to install/configure needed software.

Lets make our system up to date:

    sudo apt-get update && sudo apt-get upgrade

Now lets install software we will use (Git, compilers, archivers etc):

    sudo apt-get install git-core build-essential zlib1g-dev libssl-dev vim mc htop libmysqlclient-dev curl git-core ruby libpcre3-dev libxml2-dev libxml2 libxslt1-dev imagemagick libmagickcore-dev libmagickwand-dev autoconf byacc mysql-client libmysql-ruby libmysqlclient15-dev libmysqlclient-dev libmagickwand-dev imagemagick libmagickcore-dev zip screen curl nodejs -y 

Now we need a web server. Personally I perfer nginx. It is fast, easy to use and configure. Some one prefers installing from packages by I always do that from sources, since I can get the lates one and also I can configure modules i need. 

So lets get it(at the moment of that blog the latest stable release is 1.2.8):

    sudo useradd -s /usr/sbin/nologin nginx
    cd ~
    wget http://nginx.org/download/nginx-1.2.8.tar.gz
    tar -xvf nginx-1.2.8.tar.gz
    cd nginx-1.2.8
    ./configure --prefix='/opt/nginx' --with-http_ssl_module --with-http_gzip_static_module --with-cc-opt='-Wno-error' --with-http_stub_status_module
    make
    sudo make install
    cd ..
    rm nginx-1.2.8 -rf
    rm nginx-1.2.8.tar.gz

Few words about build configuration:

--with-http_ssl_module - Application hosted there will have ssl encryption

--with-http_stub_status_module - I will use munin nginx-* plugins for requests monitoring etc.

You can read more about nginx configuration on [official site](http://nginx.org/)

Now we need a startup script(I have a gist for that):

{% gist 5420306 %}

    cd ~
    git clone https://gist.github.com/5420306.git
    sudo cp 5420306/nginx /etc/init.d/nginx
    sudo chmod 0755 /etc/init.d/nginx
    rm 5420306 -rf

Now lets get ruby. I would recomend to use rvm even in production environment, since it will bring some flexibility to your setup. Also I recomend to use per user rvm installation rather than system wide.

     \curl -#L https://get.rvm.io | bash -s stable

Lets apply changes:
    
    source ~/.bashrc

And install latest(production) ruby version:
    
    rvm install 1.9.3-p392

I prefer to install release number rather than head. Since its quite clear how to upgrade in case of new versions.

Now gemset(I prefer to use before setup hook for capistrano) but this also will work:

    rvm use 1.9.3@appname --create

Now lets connect nginx to our rails application. We can do that using several approaches(web servers). By popularity ATM:

  * [Thin](http://code.macournoyer.com/thin/)
  * [Passenger](https://www.phusionpassenger.com/)
  * [Unicorn](http://unicorn.bogomips.org/)
  * [Pow](http://pow.cx/)
  * [Puma](http://puma.io/)
  * [Rainbows](http://rainbows.rubyforge.org/) - Unicorn fork
  * Other options

I perfer unicorn, but you might want to consider another option. Actualy I'm quite happy to have so much choices. Not every platform have such a wide range of different solutions.

Unicorn able to listen UNIX sockets, that's very handy and we can start multiple workers for our application and connect to nginx's upstream.

Our application will be located at /home/#{username}/application/#{application_name}, so socket location will be /home/#{username}/application/#{application_name}/shared/sockets/uncorn.sock. 