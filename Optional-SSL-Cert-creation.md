### If you prefer to have a valid certificate, you can SSH to Phantom box to set up certificate during setup process
   1. Run the following as root:
   ```
   yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
   yum install -y certbot python2-certbot-nginx python2-certbot-dns-google
   certbot --nginx certonly
   #Feed in your domain name and then cd to the new directory with the certs once Created
   cp /opt/phantom/etc/ssl/certs/httpd_cert.crt /opt/phantom/etc/ssl/certs/httpd_cert.crt.bak
   cp fullchain.pem /opt/phantom/etc/ssl/certs/httpd_cert.crt
   cp /opt/phantom/etc/ssl/private/httpd_cert.key /opt/phantom/etc/ssl/private/httpd_cert.key.bak
   cp privkey.pem /opt/phantom/etc/ssl/private/httpd_cert.key
   service nginx reload
   ```

   FYI: to renew certificate at a later date, simply run:
   ```
   certbot renew
   cp <path_to_new_cert>/fullchain.pem /opt/phantom/etc/ssl/certs/httpd_cert.crt
   cp <path_to_new_cert>/privkey.pem /opt/phantom/etc/ssl/private/httpd_cert.key
   service nginx reload
   ```
