# 208.4. Implementing Nginx as a web server and a reverse proxy

## **208.4 Implementing Nginx as a web server and a reverse proxy**

**Weight:** 2

**Description:** Candidates should be able to install and configure a reverse proxy server, Nginx. Basic configuration of Nginx as a HTTP server is included.

**Key Knowledge Areas:**

* Nginx
* Reverse Proxy
* Basic Web Server

**Terms and Utilities:**

* /etc/nginx/
* nginx

### Whats is nginx ?![](.gitbook/assets/nginx-logo.png)

 **NGINX**  \(short for**Engine X**\) is a free, open-source and powerful HTTP web server and reverse proxy with an event-driven \(asynchronous\) architecture. It is written using  **C**  programming language and runs on Unix-like operating systems as well as Windows OS.

It also works as a reverse proxy, standard mail and TCP/UDP proxy server, and can additionally be configured as a load balancer. It is powering many sites on the web. well known for its high-performance, stability and feature-rich set.

#### history

In 2002, Igor Sysoev start work on Nginx as a solution provider to the C10K problem, which was a extreme challenge for web servers to start handling ten thousand synchronized connections as a necessity for the modern network. The first public release of Nginx was made in 2004, meeting this goal by relying on an asynchronous, events-driven architecture.

#### nginx vs apache

Apache and Nginx both are the most common open source web servers. Together, both web servers are responsible for serving over 50% of traffic on the internet. Both solutions are capable of handling different workloads and working with other software to provide a full web stack.

While Apache and Nginx split many qualities, they should not be consideration of as completely interchangeable. Each has been developed in its own way and it is important to understand the situations where we may need to re-evaluate ourweb server of choices. This table will show how each web server stacks up in various areas:

| Criterian | Apache | Nginx |
| :--- | :--- | :--- |
| Static speed | Second to nginx | 2.5x faster than apache |
| Dynamic speed | Both same in this area | Both same in this area |
| OS support | Unix, Windows, Mac OSX | works great with Unix-like OS. Not so with windows. |
| Security | Both have excellent security track record | Both have excellent security track record |
| Flexibility | Highly customizable architecture | Difficult to customize modules for the server due to complex base architecture |
| Support | Excellent community with wide spread user base.Lots of online support | provide community  support through mailing lists, IRC , ... |
| Cost | Open source hence free to download and support | Open source license available along with paid liencese for advanced features like NGINX plus |

Comparing and getting deeper to nginx features is beyond the scope of this lesson but it is recommanded to invest some time on it.

In this course we will stablish a basic web server using nginx and then implement a revere proxy using that.

### Basic Web server

Lets start by installing Nginx web server from Ubuntu official repositories:

```text
root@server1:~# apt install nginx
```

```text
root@server1:~# systemctl status nginx.service 
● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: en
   Active: active (running) since Sat 2018-06-30 03:17:30 PDT; 2min 7s ago
 Main PID: 3525 (nginx)
   CGroup: /system.slice/nginx.service
           ├─3525 nginx: master process /usr/sbin/nginx -g daemon on; master_pro
           └─3526 nginx: worker process                           

Jun 30 03:17:30 server1 systemd[1]: Starting A high performance web server and a
Jun 30 03:17:30 server1 systemd[1]: Started A high performance web server and a 
root@server1:~# netstat -tlpen
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       User       Inode       PID/Program name
tcp        0      0 127.0.1.1:53            0.0.0.0:*               LISTEN      0          20730       1120/dnsmasq    
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN      0          38875       3375/cupsd      
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      0          39652       3525/nginx -g daemo
tcp6       0      0 ::1:631                 :::*                    LISTEN      0          38874       3375/cupsd      
tcp6       0      0 :::80                   :::*                    LISTEN      0          39653       3525/nginx -g daemo
```

### /etc/nginx/

All NGINX configuration files are located in the /etc/nginx/ directory.

```text
root@server1:~# cd /etc/nginx/
root@server1:/etc/nginx# ls -l
total 56
drwxr-xr-x 2 root root 4096 Jul 12  2017 conf.d
-rw-r--r-- 1 root root 1077 Feb 11  2017 fastcgi.conf
-rw-r--r-- 1 root root 1007 Feb 11  2017 fastcgi_params
-rw-r--r-- 1 root root 2837 Feb 11  2017 koi-utf
-rw-r--r-- 1 root root 2223 Feb 11  2017 koi-win
-rw-r--r-- 1 root root 3957 Feb 11  2017 mime.types
-rw-r--r-- 1 root root 1462 Feb 11  2017 nginx.conf
-rw-r--r-- 1 root root  180 Feb 11  2017 proxy_params
-rw-r--r-- 1 root root  636 Feb 11  2017 scgi_params
drwxr-xr-x 2 root root 4096 Jun 30 03:17 sites-available
drwxr-xr-x 2 root root 4096 Jun 30 03:17 sites-enabled
drwxr-xr-x 2 root root 4096 Jun 30 03:17 snippets
-rw-r--r-- 1 root root  664 Feb 11  2017 uwsgi_params
-rw-r--r-- 1 root root 3071 Feb 11  2017 win-utf
```

The primary configuration file is /etc/nginx/nginx.conf:

```text
root@server1:/etc/nginx# cat nginx.conf 
user www-data;
worker_processes auto;
pid /run/nginx.pid;

events {
    worker_connections 768;
    # multi_accept on;
}

http {

    ##
    # Basic Settings
    ##

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    # server_tokens off;

    # server_names_hash_bucket_size 64;
    # server_name_in_redirect off;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    ##
    # SSL Settings
    ##

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
    ssl_prefer_server_ciphers on;

    ##
    # Logging Settings
    ##

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;

    ##
    # Gzip Settings
    ##

    gzip on;
    gzip_disable "msie6";

    # gzip_vary on;
    # gzip_proxied any;
    # gzip_comp_level 6;
    # gzip_buffers 16 8k;
    # gzip_http_version 1.1;
    # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    ##
    # Virtual Host Configs
    ##

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}


#mail {
#    # See sample authentication script at:
#    # http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
# 
#    # auth_http localhost/auth.php;
#    # pop3_capabilities "TOP" "USER";
#    # imap_capabilities "IMAP4rev1" "UIDPLUS";
# 
#    server {
#        listen     localhost:110;
#        protocol   pop3;
#        proxy      on;
#    }
# 
#    server {
#        listen     localhost:143;
#        protocol   imap;
#        proxy      on;
#    }
#}
```

In CentOS the configuration file would be like this:

```text
[root@centos7-1 nginx]# cat nginx.conf
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

# Settings for a TLS enabled server.
#
#    server {
#        listen       443 ssl http2 default_server;
#        listen       [::]:443 ssl http2 default_server;
#        server_name  _;
#        root         /usr/share/nginx/html;
#
#        ssl_certificate "/etc/pki/nginx/server.crt";
#        ssl_certificate_key "/etc/pki/nginx/private/server.key";
#        ssl_session_cache shared:SSL:1m;
#        ssl_session_timeout  10m;
#        ssl_ciphers HIGH:!aNULL:!MD5;
#        ssl_prefer_server_ciphers on;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#
#        location / {
#        }
#
#        error_page 404 /404.html;
#            location = /40x.html {
#        }
#
#        error_page 500 502 503 504 /50x.html;
#            location = /50x.html {
#        }
#    }

}
```

Configuration options in NGINX are called directives. Directives are organized into groups known as blocks \(or contexts,They are synonymous\)

Lines preceded by a \# character are comments and not interpreted by NGINX. Lines containing directives must end with a ; or NGINX will fail to load the configuration and report an error.

The file starts with 5 irectives:`user`,`worker_processes`,`error_log`, and`pid`. These are outside any specific block or context, so they’re said to exist in the`main`context. See [the NGINX docs](https://nginx.org/en/docs/ngx_core_module.html) for explanations of these directives and others available in the `main`context.

The`events`and`http`blocks are areas for additional directives, and they also exist in the`main`context.

* **The http Block :**The http block contains directives for handling web traffic. These directives are often referred to as universal because they are passed on to to all website configurations NGINX serves.See [the NGINX docs](https://nginx.org/en/docs/ngx_core_module.html) for a list of available directives for the http block.
* **Server Blocks :** The`http`block above contains an`include`directive which tells NGINX where website Configuration files are located.

If we installed from the official NGINX repository, or RedHat, this line will say`include /etc/nginx/conf.d/*.conf;`as it does in the`http`block above. Each website we host with NGINX should have its own configuration file in`etc/nginx/conf.d/`, with the name formatted as`example.com.conf`. Sites which are disabled \(not being served by NGINX\) should be named `xample.com.conf.disabled`.

If we installed NGINX from the Debian or Ubuntu repositories, this line will say`include /etc/nginx/sites-enabled/*;`. The`../sites-enabled/`folder contains symlinks to the site configuration files stored in`/etc/nginx/sites-available/`. Sites in`sites-available`can be disabled by removing the symlink to`sites-enabled`.

```text
root@server1:/etc/nginx# cd sites-enabled/
root@server1:/etc/nginx/sites-enabled# ls -l
total 0
lrwxrwxrwx 1 root root 34 Jun 30 03:17 default -> /etc/nginx/sites-available/default

root@server1:/etc/nginx/sites-enabled# cd ../sites-available/
root@server1:/etc/nginx/sites-available# ls -l
total 4
-rw-r--r-- 1 root root 2074 Feb 11  2017 default

root@server1:/etc/nginx/sites-available# cat default 
##
# You should look at the following URL's in order to grasp a solid understanding
# of Nginx configuration files in order to fully unleash the power of Nginx.
# http://wiki.nginx.org/Pitfalls
# http://wiki.nginx.org/QuickStart
# http://wiki.nginx.org/Configuration
#
# Generally, you will want to move this file somewhere, and start with a clean
# file but keep this around for reference. Or just disable in sites-enabled.
#
# Please see /usr/share/doc/nginx-doc/examples/ for more detailed examples.
##

# Default server configuration
#
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    # SSL configuration
    #
    # listen 443 ssl default_server;
    # listen [::]:443 ssl default_server;
    #
    # Note: You should disable gzip for SSL traffic.
    # See: https://bugs.debian.org/773332
    #
    # Read up on ssl_ciphers to ensure a secure configuration.
    # See: https://bugs.debian.org/765782
    #
    # Self signed certs generated by the ssl-cert package
    # Don't use them in a production server!
    #
    # include snippets/snakeoil.conf;

    root /var/www/html;

    # Add index.php to the list if you are using PHP
    index index.html index.htm index.nginx-debian.html;

    server_name _;

    location / {
        # First attempt to serve request as file, then
        # as directory, then fall back to displaying a 404.
        try_files $uri $uri/ =404;
    }

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    include snippets/fastcgi-php.conf;
    #
    #    # With php7.0-cgi alone:
    #    fastcgi_pass 127.0.0.1:9000;
    #    # With php7.0-fpm:
    #    fastcgi_pass unix:/run/php/php7.0-fpm.sock;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny all;
    #}
}


# Virtual Host configuration for example.com
#
# You can move that to a different file under sites-available/ and symlink that
# to sites-enabled/ to enable it.
#
#server {
#    listen 80;
#    listen [::]:80;
#
#    server_name example.com;
#
#    root /var/www/example.com;
#    index index.html;
#
#    location / {
#        try_files $uri $uri/ =404;
#    }
#}
```

Regardless of the installation source, server configuration files will contain as`server`block \(or blocks\) for a website.

* **Listening ports :** The `listen`directive tells NGINX the hostname/IP and the TCP port where it should listen for HTTP connections. The argument `default_server` means this virtual host will answer requests on port 80 that don’t specifically match another virtual host’s listen statement. The second statement listens over IPv6 and behaves similarly.
* **Name-based virtual Hosting :** The `server_name` directive allows multiple domains to be served from a single IP address. The server decides which domain to serve based on the request header it receives.

```text
     #examples: 
              #process requests for two addresses:
              serve_name       example.com www.example.com;
              # using wildcrads
              server_name       *.example.com;
              server_name       .example.com;
              #Process requests for all domain names beginning with example.
              server_name       example.*;
```

NGINX allows us to specify server names that are not valid domain names. NGINX uses the name from the HTTP header to answer requests, regardless of whether the domain name is valid or not.

Using non-domain hostnames is useful if our server is on a LAN, or if we already know all of the clients that will be making requests of the server.

* **Location Blocks :** The `location`setting lets we configure how NGINX will respond to requests for resources within the server. Just like the `server_name`directive tells NGINX how to process requests for the domain, `location`directives cover requests for specific files and folders, such as `http://example.com/blog/`.  `location`also supports regular expressions:

```text
    ~* : matches to be case-insensitive
    ^~ :tells NGINX, if it matches a particular string, to stop searching for more specific matches and use the directives here instead. Other than that, these directives work like the literal string matches in the first group. Even if there’s a more specific match later, if a request matches one of these directives, the settings here will be used. See below for more information about the order and priority of location directive processing.
    =  : this forces an exact match with the path requested and then stops searching for more specific matches.
```

* **Location Root and Index:** The`location`setting is another variable that has its own block of arguments.

  Once NGINX has determined which`location`directive best matches a given request, the response to this request is determined by the contents of the associated`location`directive block. Here’s an example:

```text
location / {
    root html;
    index index.html index.htm;
}
```

In this example, the document root is located in the`html/`directory. Under the default installation prefix for NGINX, the full path to this location is`/etc/nginx/html/`.The`index`variable tells NGINX which file to serve if none is specified.

### /usr/share/ngnix/html

The default public web root in NGNIX is `/usr/share/ngnix/html` how ever, here in ubuntu16.04 the default web root is:

```text
root@server1:/etc/nginx/sites-available# cat default | grep root
    root /var/www/html;
    # deny access to .htaccess files, if Apache's document root
#    root /var/www/example.com;
root@server1:/etc/nginx/sites-available# cd /var/www/html/
root@server1:/var/www/html# ls -l
total 4
-rw-r--r-- 1 root root 612 Jun 30 03:17 index.nginx-debian.html
```

Lets create ourselves index.html :

```text
root@server1:/var/www/html# mv index.nginx-debian.html index.nginx-debian.bak
root@server1:/var/www/html# vim index.html
root@server1:/var/www/html# cat index.html 
<h1> example site powered by NGINX ! </h1>
root@server1:/var/www/html# systemctl restart nginx.service
```

and lets chek the results:

```text
root@server1:/var/www/html# telnet localhost 80
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
GET /index.html
<h1> example site powered by NGINX ! </h1>
Connection closed by foreign host.
```

## nginx command

nginx deamon has a handy command tool `nginx`which can help us save more time, see some of switches:

```text
root@server1:~# nginx -?
nginx version: nginx/1.10.3 (Ubuntu)
Usage: nginx [-?hvVtTq] [-s signal] [-c filename] [-p prefix] [-g directives]

Options:
  -?,-h         : this help
  -v            : show version and exit
  -V            : show version and configure options then exit
  -t            : test configuration and exit
  -T            : test configuration, dump it and exit
  -q            : suppress non-error messages during configuration testing
  -s signal     : send signal to a master process: stop, quit, reopen, reload
                    stop — shut down quickly
                    quit — shut down gracefully
                    reload — reload configuration, start the new worker process with a new configuration, gracefully shut down old worker processes.
                    reopen — reopen log files
  -p prefix     : set prefix path (default: /usr/share/nginx/)
  -c filename   : set configuration file (default: /etc/nginx/nginx.conf)
  -g directives : set global directives out of configuration file
```

## Reverse Proxy

ngnix can be configured to works as a Reversy Proxy. It seats some where between client and web server and can do caching, loadbalancing, acceleration, client Authentication, bandwidth management and etc.

The most significant beneit of using proxies in our networks is server abstaraction. Becuase the client sees the proxy and not the actual server itself. From other side, when the server respond to the client request,it responds to the proxy , which respond to the client. This way we have added an abstaraction layer between client and the server. Client just sees the proxy server and doesn't have any idea about server configuration and on the other hand server is protected from evil hackers.

Working as a ReverseProxy is what ngnix is famous for and configuring it is so simple.It is configured as like as a virtual host.So lets start, we use Ubuntu and use previous basic web server. we have setup to proxy example.com:

```text
root@server1:~# vim /etc/hosts
root@server1:~# cat /etc/hosts
127.0.0.1    localhost
127.0.1.1    ubuntu
127.0.0.1    example.com
# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
root@server1:~# ping example.com 
PING example.com (127.0.0.1) 56(84) bytes of data.
64 bytes from localhost (127.0.0.1): icmp_seq=1 ttl=64 time=0.017 ms
64 bytes from localhost (127.0.0.1): icmp_seq=2 ttl=64 time=0.098 ms
64 bytes from localhost (127.0.0.1): icmp_seq=3 ttl=64 time=0.076 ms
^C
--- example.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2034ms
rtt min/avg/max/mdev = 0.017/0.063/0.098/0.035 ms
```

Now we want to proxy request for example.com to the google.com, for that we need some configurations.

```text
root@server1:~# cd /etc/nginx/
root@server1:/etc/nginx# ls -l
total 56
drwxr-xr-x 2 root root 4096 Jul 12  2017 conf.d
-rw-r--r-- 1 root root 1077 Feb 11  2017 fastcgi.conf
-rw-r--r-- 1 root root 1007 Feb 11  2017 fastcgi_params
-rw-r--r-- 1 root root 2837 Feb 11  2017 koi-utf
-rw-r--r-- 1 root root 2223 Feb 11  2017 koi-win
-rw-r--r-- 1 root root 3957 Feb 11  2017 mime.types
-rw-r--r-- 1 root root 1462 Feb 11  2017 nginx.conf
-rw-r--r-- 1 root root  180 Feb 11  2017 proxy_params
-rw-r--r-- 1 root root  636 Feb 11  2017 scgi_params
drwxr-xr-x 2 root root 4096 Jul  1 22:33 sites-available
drwxr-xr-x 2 root root 4096 Jun 30 03:17 sites-enabled
drwxr-xr-x 2 root root 4096 Jun 30 03:17 snippets
-rw-r--r-- 1 root root  664 Feb 11  2017 uwsgi_params
-rw-r--r-- 1 root root 3071 Feb 11  2017 win-utf
```

The simplest and minimal configuration which are requred for setting up a reverse proxy using nginx is inside `proxy_param` :

```text
root@server1:/etc/nginx# cat proxy_params 
proxy_set_header Host $http_host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
```

We need weather copy these configuration inside our virtual host or as we do here, include it.

```text
root@server1:/etc/nginx# cd sites-available/
root@server1:/etc/nginx/sites-available# ls -l
total 4
-rw-r--r-- 1 root root 2074 Feb 11  2017 default
root@server1:/etc/nginx/sites-available# vim example.com
root@server1:/etc/nginx/sites-available# cat example.com 
server {
    listen 80;
    server_name example.com;

    location \ {
        proxy_pass http://lxer.com/;
        include /etc/nginx/proxy_params;
        }
    }
```

Now lets make it enable and restart the nginx service:

```text
root@server1:/etc/nginx/sites-available# cd ../sites-enabled/
root@server1:/etc/nginx/sites-enabled# ln -s ../sites-available/example.com .
root@server1:/etc/nginx/sites-enabled# ls -l
total 0
lrwxrwxrwx 1 root root 34 Jun 30 03:17 default -> /etc/nginx/sites-available/default
lrwxrwxrwx 1 root root 30 Jul  1 22:48 example.com -> ../sites-available/example.com

root@server1:/etc/nginx/sites-enabled# systemctl restart nginx.service 
root@server1:/etc/nginx/sites-enabled# systemctl status nginx.service 
● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2018-07-01 22:52:04 PDT; 8s ago
  Process: 6867 ExecStop=/sbin/start-stop-daemon --quiet --stop --retry QUIT/5 --pidfile /run/nginx.pid (code=exited, status=0/SUCCESS)
  Process: 6937 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
  Process: 6935 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
 Main PID: 6940 (nginx)
   CGroup: /system.slice/nginx.service
           ├─6940 nginx: master process /usr/sbin/nginx -g daemon on; master_process on
           └─6941 nginx: worker process                           

Jul 01 22:52:03 server1 systemd[1]: Starting A high performance web server and a reverse proxy server...
Jul 01 22:52:04 server1 systemd[1]: nginx.service: Failed to read PID from file /run/nginx.pid: Invalid argument
Jul 01 22:52:04 server1 systemd[1]: Started A high performance web server and a reverse proxy server.
```

Now it is time to check the results:

```text
root@server1:/etc/nginx/sites-enabled# elinks http://example.com
```

and see what will happend.

