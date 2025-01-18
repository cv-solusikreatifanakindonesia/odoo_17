### INTEGRATE NGINX AND APACHE2 for ODOO APPS WITH NGINX and Other Domain Apache2 Following WHM & CPANEL WITH SUPPORT LIVE CHAT
## EXAMPLEODOOAPPS.COM Nginx Conf as point for default domain via nginx installed as standalone with port 443 for ssl and 80 for non ssl | Example Conf

## TESTED AND WORKS ON WINDOWS SERVER 2022 ##

upstream odoo {
  server 127.0.0.1:8070;
}

server {
  listen 80;
  server_name exampleodooapps.com;
  rewrite ^(.*) https://exampleodooapps.com/$1 permanent;
}

server {
  listen 443 ssl;
  server_name exampleodooapps.com;

  proxy_buffers 16 64k;
  proxy_buffer_size 128k;
  proxy_connect_timeout 3600;
  proxy_send_timeout 3600s;
  proxy_read_timeout 3600s;
  proxy_next_upstream error timeout invalid_header http_500 http_502 http_503;

  # SSL parameters
  ssl_certificate C:/SSL_Certificates/exampleodooapps.com/exampleodooapps.com.crt;
  ssl_certificate_key C:/SSL_Certificates/exampleodooapps.com/exampleodooapps.com.key;

  # log
  access_log C:/inetpub-erps/public/exampleodooapps_com_access_https.log;
  error_log C:/inetpub-erps/public/exampleodooapps_com_errors_https.log;

  # Redirect websocket requests to odoo gevent port
  location /websocket {
	proxy_http_version 1.1;
	proxy_pass http://127.0.0.1:8070/websocket;
	proxy_set_header Upgrade $http_upgrade;
	proxy_set_header Connection "upgrade";
  }

  # Redirect requests to odoo backend server
  location / {
	# Add Headers for odoo proxy mode
	proxy_set_header X-Forwarded-Host $http_host;
	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	proxy_set_header X-Forwarded-Proto $scheme;
	proxy_set_header X-Real-IP $remote_addr;
	proxy_redirect off;
	proxy_pass http://odoo;

	add_header 'Access-Control-Allow-Origin' '*' always;
	add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS, HEAD';
	add_header 'Access-Control-Allow-Headers' 'Authorization, Origin, X-Requested-With, Content-Type, Accept';
	add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";
  }

  # common gzip
  gzip_types text/css text/scss text/plain text/xml application/xml application/json application/javascript;
  gzip on;
  client_body_in_file_only clean;
  client_body_buffer_size 32K;
  client_max_body_size 500M;
  sendfile on;
  send_timeout 3600s;
  keepalive_timeout 3600;
}

## END ##
## TESTED AND WORKS ON ALMALINUX 8 ##
# CUSTOM ODOO NGINX CONFIG

upstream odoo {
  server 127.0.0.1:8070;
}
upstream odoochat {
  server 127.0.0.1:8072;
}
map $http_upgrade $connection_upgrade {
  default upgrade;
  ''      close;
}

# http -> https
server {
  listen 80;
  server_name exampleodooapps.com;
  rewrite ^(.*) https://exampleodooapps.com/$1 permanent;
}

server {
  listen 443 ssl;
  server_name exampleodooapps.com;

  proxy_buffers 16 64k;
  proxy_buffer_size 128k;
  proxy_connect_timeout 600s;
  proxy_send_timeout 600s;
  proxy_read_timeout 600s;
  proxy_next_upstream error timeout invalid_header http_500 http_502 http_503;

  # SSL parameters
  ssl_certificate /home/usercpanel/ssl/certs/sslcertfiles.crt;
  ssl_certificate_key /home/usercpanel/ssl/keys/sslkeyfiles.key;

  # log
  access_log /home/usercpanel/public_html/exampleodooapps.com/access_https.log;
  error_log /home/usercpanel/public_html/exampleodooapps.com/errors_https.log;

  # Redirect websocket requests to odoo gevent port
  location /websocket {
    proxy_pass http://odoochat;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;
    proxy_set_header X-Forwarded-Host $http_host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Real-IP $remote_addr;

    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";
  }

  # Redirect requests to odoo backend server
  location / {
    # Add Headers for odoo proxy mode
    proxy_set_header X-Forwarded-Host $http_host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_redirect off;
    proxy_pass http://odoo;

    add_header 'Access-Control-Allow-Origin' '*' always;
    add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS, HEAD';
    add_header 'Access-Control-Allow-Headers' 'Authorization, Origin, X-Requested-With, Content-Type, Accept';
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";
  }

  # common gzip
  gzip_types text/css text/scss text/plain text/xml application/xml application/json application/javascript;
  gzip on;
  client_body_in_file_only clean;
  client_body_buffer_size 32K;
  client_max_body_size 500M;
  sendfile on;
  send_timeout 600s;
  keepalive_timeout 300;
}
## END ##

## otherdomainapachecpanel.com VirtualHost as default domain via apache in whm with port 8080 for Non SSL and 8443 for SSL | Example Conf

server {
    listen 80;
    server_name otherdomainapachecpanel.com www.otherdomainapachecpanel.com;

    # Redirect all HTTP traffic to HTTPS
    rewrite ^(.*) https://otherdomainapachecpanel.com/$1 permanent;
}

server {
    listen 443 ssl;
    server_name otherdomainapachecpanel.com www.otherdomainapachecpanel.com;

    # SSL configuration
    ssl_certificate /home/usercpanel/ssl/certs/sslcertfiles.crt;
    ssl_certificate_key /home/usercpanel/ssl/keys/sslkeyfiles.key;

    # Proxy pass to Apache on port 8443 for SSL
    location / {
        proxy_pass https://otherdomainapachecpanel.com:8443;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# Apache config for HTTP on port 8080
<VirtualHost otherdomainapachecpanel.com:8080>
  ServerName otherdomainapachecpanel.com
  DocumentRoot /home/usercpanel/public_html/otherdomainapachecpanel.com

  ErrorLog /home/usercpanel/public_html/otherdomainapachecpanel.com/errors_http.log
</VirtualHost>

# Apache config for HTTPS on port 8443
<VirtualHost otherdomainapachecpanel.com:8443>
  ServerName otherdomainapachecpanel.com
  DocumentRoot /home/usercpanel/public_html/otherdomainapachecpanel.com

  SSLEngine on
  SSLCertificateFile /home/usercpanel/ssl/certs/sslcertfiles.crt
  SSLCertificateKeyFile /home/usercpanel/ssl/keys/sslkeyfiles.key

  ErrorLog /home/usercpanel/public_html/otherdomainapachecpanel.com/errors_https.log
</VirtualHost>

Browse to "WHM >> Tweak Settings" under the "Security" 
Updating “Enable File Protect” from “On” to “Off”.

Browse to "WHM >> Tweak Settings" under the "System" 
Updating “Port to” from “8080 for non SSL && 8443 for SSL”.


## Don't forget to set the domain in Settings > Website > name exampleodooapps.com, and for the live chat in the website header > top right corner, click Edit > Theme Tab > Code Injection. Insert the HTML below and adjust accordingly.
<script type="text/javascript" src="https://exampleodooapps.com/im_livechat/loader/[following id channel]"></script>
<script type="text/javascript" src="https://exampleodooapps.com/im_livechat/assets_embed.js"></script>
## And in Setting > Technical > System Parameters make sure the variable web.base.url is https://exampleodooapps.com

#Before Start for Mac
brew install --cask wkhtmltopdf && pip3 install setuptools wheel && pip3 install -r requirements.txt
#Before Start for AlmaLinux
sudo dnf install -y python3-pip  && pip3 install setuptools wheel && pip3 install -r requirements.txt

#first step
export TMPDIR=/tmp && export TMP_DIR=/tmp
#second step for almalinux install wkhtmltopdf
https://computingforgeeks.com/install-wkhtmltopdf-wkhtmltoimage-on-rocky-almalinux/

#Basic Config
## TESTED AND WORKS ON ALMALINUX 8 ##
[options]
; Basic configuration
db_host = [database hostname]
db_port = [database port]
db_name = [database name] 
db_user = [database username]
db_password = [database password]
addons_path = addons
xmlrpc_port = 8069
proxy_mode = True
without_demo = True
admin_passwd = [password for backup database or master password]
limit_memory_hard = 2415919104
limit_memory_soft = 2013265920
limit_request = 8192
limit_time_cpu = 360
limit_real_time = 3600
limit_time_real_cron = 2
gevent_port=8072
max_cron_threads = 1
workers = 3
websocket_keep_alive_timeout = 600
websocket_rate_limit_burst = 10
websocket_rate_limit_delay = 0.2
xmlrpc = True
logfile = ../odoo_erp_logs.log
## END ##

## TESTED AND WORKS ON WINDOWS SERVER 2022 ##
[options]
admin_passwd = [password for backup database or master password]
db_host = [database hostname]
db_port = [database port]
db_name = [database name] 
db_user = [database username]
db_password = [database password]
addons_path = addons,custom_addons,custom_themes,custom_backend_themes/muk
xmlrpc_port = 8069
proxy_mode = True
without_demo = True
limit_memory_hard = 2415919104
limit_memory_soft = 2013265920
limit_request = 8192
limit_time_cpu = 360
limit_real_time = 3600
limit_time_real_cron = 2
gevent_port=8069
max_cron_threads = 2
workers = 2
websocket_keep_alive_timeout = 600
websocket_rate_limit_burst = 10
websocket_rate_limit_delay = 0.2
xmlrpc = True
logfile = ../odoo_erp_logs.log
## END ##


##START PROGRAM for after install base
./odoo-bin -c config.conf

##START PROGRAM FOR FIRST for Install Base
./odoo-bin -c config.conf -i base

##ISSUE FIXING

#IF Database Cannot Remove
SELECT pid, usename, application_name, client_addr, backend_start
FROM pg_stat_activity
WHERE datname = '[database name]';

SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE datname = '[database name]'
  AND pid <> pg_backend_pid();
 
DROP DATABASE [database name];

#Error Symbol not found: _PQbackendPID
https://gist.github.com/peter-gy/0ebe072acb9065944ecb04d95a4c3096
