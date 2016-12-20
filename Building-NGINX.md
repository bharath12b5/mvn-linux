<!---PACKAGE:NGINX--->
<!---DISTRO:RHEL 6.x:1.10--->
<!---DISTRO:RHEL 7.x:1.10--->
<!---DISTRO:SLES 11.x:1.10--->
<!---DISTRO:SLES 12.x:1.10--->
<!---DISTRO:Ubuntu 16.x:1.10--->

# Building NGINX

Below versions of NGINX are available in respective distributions at the time of this recipe creation:

*    Ubuntu 16.04 has `1.10.0`

The instructions provided below specify the steps to build NGINX version 1.10.2 on the IBM z Systems for RHEL 6.8/7.1/7.2/7.3, SLES 11-SP4/12/12-SP1/12-SP2 and Ubuntu 16.04/16.10.


_**General Notes:**_ 	 
 * When following the steps below please use a standard permission user unless otherwise specified._

 * A directory `/<source_root>/` will be referred to in these instructions, this is a temporary writeable directory anywhere you'd like to place it._

### Downloading, Building and Installing NGINX 1.10.2

1.  Install dependencies as needed for the specific platform

    RHEL 6.8

    ```source-shell
    sudo yum -y install pcre-devel wget tar xz gcc make zlib-devel
    ```
    RHEL 7.1/7.2/7.3

    ```source-shell
    sudo yum -y install pcre-devel wget tar gcc make zlib-devel
    ```

    SLES 11-SP4/12/12-SP1/12-SP2

    ```source-shell
    sudo zypper install -y pcre-devel wget tar gcc make zlib-devel
    ```
	
    Ubuntu 16.04/16.10

    ```source-shell
	sudo apt-get update
    sudo apt-get install -y  wget tar gcc make libpcre3-dev openssl libssl-dev
    ``` 
    

2.  Download and unpack the NGINX 1.10.2 source package

    ```source-shell
	mkdir /<source_root>/
    cd /<source_root>/
    wget http://nginx.org/download/nginx-1.10.2.tar.gz
    tar xvf nginx-1.10.2.tar.gz
    cd nginx-1.10.2
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

4.  **(Optional)** NGINX will be installed in /usr/local/nginx/sbin/; depending upon user preferences and conventions, it may be necessary to either update PATH or create links to the executable files

### Simple Proxy Test

1.  Prepare a test webpage `index.html` to serve in `/tmp/` folder.

    For example, this simple HTML document provides a bare minimum of text:

    ```text-html-basic
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
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