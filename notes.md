##VirtualHost Example Conf
<VirtualHost example.com:80>
  ServerName example.com
  ProxyRequests Off
  ProxyPreserveHost On
  ProxyPass / http://ipaddress_or_hostname_server:8069/
  ProxyPassReverse / http://ipaddress_or_hostname_server:8069/
  RequestHeader set X-Forwarded-Proto "https" early
  ErrorLog /homepath/usercpanel/public_html/example.com/errors_http.log
</VirtualHost>

<VirtualHost example.com:443>
  ServerName example.com
  ProxyRequests Off
  ProxyPreserveHost On
  RequestHeader set X-Forwarded-Proto "https" early
  SSLEngine on
  SSLCertificateFile /homepath/usercpanel/ssl/certs/certname.crt
  SSLCertificateKeyFile /homepath/usercpanel/ssl/keys/keyname.key

  ProxyPass / http://ipaddress_or_hostname_server:8069/
  ProxyPassReverse / http://ipaddress_or_hostname_server:8069/
  <Directory /homepath/usercpanel/public_html/example.com>
     Order Allow,Deny
     Allow from all
     AllowOverride all
     Header set Access-Control-Allow-Origin "*"
  </Directory>
  ErrorLog /homepath/usercpanel/public_html/example.com/errors_https.log
</VirtualHost>

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
limit_memory_hard = 0
proxy_mode = True

; Uncomment the following lines if you want to enable logging
logfile = ../odoo_erp_logs.log

##START PROGRAM
./odoo-bin -c config.conf

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
