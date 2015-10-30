**General Notes**: NGINX version 1.6.3 has been successfully built and tested for RHEL 6.6/7 and SLES 11/12. When following the steps below please use a standard permission user unless otherwise specified.

## Downloading, Building and Installing NGINX 1.6.3
1. Install dependencies as needed for the specific platform

    RHEL 6.6/7
    ```shell
    sudo yum -y install pcre-devel wget
    ```
   
    SLES 11/12
    ```shell
    sudo zypper install pcre-devel wget
    ```

2. Download and unpack the NGINX 1.6.3 source package

    ```shell
    wget http://nginx.org/download/nginx-1.6.3.tar.gz
    tar xvf nginx-1.6.3.tar.gz
    cd nginx-1.6.3
    ```
   
    Alternatively, if the latest development code is preferred *(requires Mercurial)*:
    ```shell
    hg clone http://hg.nginx.org/nginx/
    cd nginx
    ```
   
3. Build and install NGINX
    ```shell
    ./configure
    ```
   
    Alternatively, if using development source:
    ```shell
    ./auto/configure
    ```
   
    Once configured:
    ```shell
    make
    sudo make install
    ```

4. **(Optional)** NGINX will be installed in /usr/local/nginx/sbin/; depending upon user preferences and conventions, it may be necessary to either update PATH or create links to the executable files.

## Simple Proxy Test

1. Prepare an test webpage to serve.

   For example, this simple HTML document provides a bare minimum of text:
   ```HTML
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
2. Create a file ```nginx.conf``` with following contents

   This is a simple test of NGINX's proxy functionality, with NGINX serving as a proxy between an HTTP user and the webpage:
   ```shell
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
   This assumes that the webpage from step 1 is stored in ```/tmp/index.html```. If this is not the case, modify the definitions for ```root``` and ```index``` accordingly.

3. Run Nginx with the provided configuration.
   ```shell
   /usr/local/nginx/sbin/nginx -c /tmp/myconfig.conf
   ```
   Note that this will normally need to be done as root, or Nginx will not have authority to access one or more ports, such as 80 and 8080.

4. Navigate a web browser to the address of the server running NGINX. The browser should display the webpage specified in ```haproxy.config```.

## References:
* [NGINX Community](http://wiki.nginx.org/Main)