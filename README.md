# docker-php-apache-separately-setup
Set up Apache and PHP Docker images to work together

Many people encounter issue when trying to set up Apache and PHP as two separate Docker images. This will be a short example demonstrating how to connect a PHP image to an Apache HTTP Server (httpd) image and successfully open a PHP file in a web browser.

For this example, I will pull two Docker images: [php:7.3.0-fpm-alpine](https://hub.docker.com/_/php) and [httpd:2.4.58-alpine](https://hub.docker.com/_/httpd).
Apache will store project files in ***/usr/local/apache2/htdocs***.
The Apache HTTP Server (httpd) configuration will look like this:

```
Listen 80
ServerName localhost

<VirtualHost *:80>
    DocumentRoot "/usr/local/apache2/htdocs/"
    ProxyPassMatch ^/(.*\.php(/.*)?)$ fcgi://myphp:9000/var/www/html/$1
    
    <Directory "/usr/local/apache2/htdocs/">
        DirectoryIndex index.php
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```
It will be copied to the appropriate location using the **docker-compose.yml** file. Take a note that we include **myphp** service and port **9000** as default port for this image.


The key point is to enable the following two directives:

```
LoadModule proxy_module modules/mod_proxy.so 
LoadModule proxy_fcgi_module modules/mod_proxy_fcgi.so
```
The directive `mod_proxy`  will ensure proper communication between the Apache and PHP Docker images. This module implements a proxy and caching server, allowing HTTP/HTTPS requests to be forwarded to other servers, in this case to php backend server. It works in conjunction with the `mod_proxy_fcgi` directive to handle requests for PHP files and pass them to the PHP-FPM server.


We will take advantage of Docker Compose to build and run our containers using this command:

`docker-compose up -d --build`

This will ensure that our Apache and PHP Docker images work together.
