# Building NGINX

NGINX version 1.8.0 has been successfully built and tested for RHEL 6.6/7.1 and SLES 11/12.

_**General Notes:**_ 	 
_i) When following the steps below please use a standard permission user unless otherwise specified._

_ii) A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._


## Downloading, Building and Installing NGINX 1.8.0

1.  Install dependencies as needed for the specific platform

    RHEL 6.6

    ```source-shell
    sudo yum -y install pcre-devel wget tar xz gcc make zlib-devel
    ```
    RHEL 7.1

    ```source-shell
    sudo yum -y install pcre-devel wget tar gcc make zlib-devel
    ```

    SLES 11/12

    ```source-shell
    sudo zypper install -y pcre-devel wget tar gcc make zlib-devel
    ```
    

2.  Download and unpack the NGINX 1.8.0 source package

    ```source-shell
	mkdir /<source_root>/
    cd /<source_root>/
    wget http://nginx.org/download/nginx-1.8.0.tar.gz
    tar xvf nginx-1.8.0.tar.gz
    cd nginx-1.8.0
    ```

3.  Build and install NGINX

    ```source-shell
    ./configure
    ```
    
    Once configured:

    ```source-shell
    make
    sudo make install
    ```

4.  **(Optional)** NGINX will be installed in /usr/local/nginx/sbin/; depending upon user preferences and conventions, it may be necessary to either update PATH or create links to the executable files.

## Simple Proxy Test

1.  Prepare a test webpage `index.html` to serve in `/tmp/` folder.

    For example, this simple HTML document provides a bare minimum of text:

    ```text-html-basic
    DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
    <html>
    <head>
     <title>Test HTML File</title>
     <meta http-equiv="Content-Type" content="text/html;charset=utf-8" />
    </head>
    <body>
     <p>This is a very simple HTML file.</p>
    </body>
    </html>
    ```

2.  Create a file `nginx.conf` with following contents in `/tmp/` folder.

    This is a simple test of NGINX's proxy functionality, with NGINX serving as a proxy between an HTTP user and the webpage:

    ```source-shell
    worker_processes     3;
    error_log            logs/error.log;
    pid                  logs/nginx.pid;
    worker_rlimit_nofile 8192;

    events
    {
    worker_connections  2048;
    }

    http
    {
    index index.html index.htm index.php;

    default_type application/octet-stream;
    log_format   main '$remote_addr - $remote_user [$time_local]  $status '
       '"$request" $body_bytes_sent "$http_referer" '
       '"$http_user_agent" "$http_x_forwarded_for"';
    access_log   logs/access.log  main;
    sendfile     on;
    tcp_nopush   on;
    server_names_hash_bucket_size 128;

    server
    {
     listen 8080;
     root /tmp;
     location /
     {
     }
    }
    server {
     location / {
     proxy_pass http://localhost:8080/;
     }
    }
    }
    ```

    This assumes that the webpage from step 1 is stored in `/tmp/index.html`. If this is not the case, modify the definitions for `root` and `index` accordingly.

3.  Run NGINX with the provided configuration.

    ```source-shell
    /usr/local/nginx/sbin/nginx -c /tmp/nginx.conf
    ```

    Note that this will normally need to be done as root, or NGINX will not have authority to access one or more ports, such as 80 and 8080.

4.  Navigate a web browser to the address of the server running NGINX. The browser should display the webpage specified in `index.html`.

## [<span class="octicon octicon-link"></span>](#references)References:

*   [NGINX Community](http://wiki.nginx.org/Main)