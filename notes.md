##VirtualHost as default domain via apache in whm | Example Conf
<VirtualHost example.com:443>
  ServerName example.com
  ProxyRequests On
  ProxyPreserveHost On
  SSLProxyEngine on
  RequestHeader set X-Forwarded-Proto "https" early
  SSLEngine on
  SSLCertificateFile /home/usercpanel/ssl/certs/sslcertificate_name.crt
  SSLCertificateKeyFile /home/usercpanel/ssl/keys/sslcertificatekey_name.key

  ProxyPass / https://example.com:8443/
  ProxyPassReverse / https://example.com:8443/

  # Additional headers
  Header add X-Forwarded-Host %{HTTP_HOST}s
  Header add X-Forwarded-For %{REMOTE_ADDR}s
  Header add X-Forwarded-Proto https
  Header add X-Real-IP %{REMOTE_ADDR}s
  # Set CORS headers for the response from the proxy
  Header set Access-Control-Allow-Origin "*"
  Header set Access-Control-Allow-Methods "GET, POST, OPTIONS"
  Header set Access-Control-Allow-Headers "Origin, X-Requested-With, Content-Type, Accept, Authorization"
  Header set Access-Control-Allow-Credentials "true"

  # Allow CORS preflight requests
  Header always set Access-Control-Allow-Origin "*"
  Header always set Access-Control-Allow-Methods "GET, POST, OPTIONS"
  Header always set Access-Control-Allow-Headers "Origin, X-Requested-With, Content-Type, Accept, Authorization"
  Header always set Access-Control-Allow-Credentials "true"

  # Handle preflight OPTIONS requests (CORS preflight)
  SetEnvIf Request_Method OPTIONS IS_OPTIONS
  Header always set Access-Control-Allow-Origin "*"
  Header always set Access-Control-Allow-Methods "GET, POST, OPTIONS"
  Header always set Access-Control-Allow-Headers "Origin, X-Requested-With, Content-Type, Accept, Authorization"
  Header always set Access-Control-Allow-Credentials "true"

  <Directory /home/usercpanel/public_html/example.com>
     Order Allow,Deny
     Allow from all
     AllowOverride all
     Header set Access-Control-Allow-Origin "*"
  </Directory>

  ErrorLog /home/usercpanel/public_html/example.com/errors_https.log
</VirtualHost>

##Nginx Conf as point for default domain via nginx installed as standalone with port 8443 for ssl and 8080 for non ssl | Example Conf
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
  listen 8080;
  server_name example.com;
  
  rewrite ^(.*) https://$host$1 permanent;
}

server {
  listen 8443 ssl;
  server_name example.com;

  proxy_buffers 16 64k;
  proxy_buffer_size 128k;
  proxy_connect_timeout 600s;
  proxy_send_timeout 600s;
  proxy_next_upstream error timeout invalid_header http_500 http_502 http_503;

  # SSL parameters
  ssl_certificate /home/usercpanel/ssl/certs/sslcertificate_name.crt;
  ssl_certificate_key /home/usercpanel/ssl/keys/sslcertificatekey_name.key;

  # log
  access_log /home/usercpanel/public_html/example.com/access_https.log;
  error_log /home/usercpanel/public_html/example.com/errors_https.log;

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

#Before Start for Mac
brew install --cask wkhtmltopdf && pip3 install setuptools wheel && pip3 install -r requirements.txt
#Before Start for AlmaLinux
sudo dnf install -y python3-pip  && pip3 install setuptools wheel && pip3 install -r requirements.txt

#first step
export TMPDIR=/tmp && export TMP_DIR=/tmp
#second step for almalinux install wkhtmltopdf
https://computingforgeeks.com/install-wkhtmltopdf-wkhtmltoimage-on-rocky-almalinux/

#Basic Config
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

##START PROGRAM
./odoo-bin -c config.conf

##START PROGRAM FOR FIRST 
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
