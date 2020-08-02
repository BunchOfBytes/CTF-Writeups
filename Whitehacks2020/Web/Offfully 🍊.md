# Offfully 🍊
### Summary 
Misconfigured NGINX server allows for path traversal -> Flag

### Solution
We are given a configuration file for NGINX and a web server to pwn.

``````
server {
        listen 80 default_server;
        listen [::]:80 default_server;
        root /var/www/html;
        index index.html index.htm index.nginx-debian.html;
        server_name _;
        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                try_files $uri $uri/ =404;
        }
        location = /flag.txt {
                deny all;
                return 403;
        }
        location /images {
                alias /var/www/html/images/;
        }
}
``````

When we visit /flag.txt we get a 403 Forbidden error. However, when we visit /images we get pointed to /var/www/html/images/ part of the directory. According to this post https://www.acunetix.com/vulnerabilities/web/path-traversal-via-misconfigured-nginx-alias/

That means that when I visit /images../flag.txt, it translates to /var/www/html/images/../flag.txt. This will give us a flag.txt

Flag: WH2020{0FF_BY_TW0_D0Ts_AND_A_SLASH}
