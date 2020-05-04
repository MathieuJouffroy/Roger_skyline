## You have to set a web server who should BE available on the VM’s IP or an host (init.login.com for exemple). About the packages of your web server, you can choose between Nginx and Apache. You have to set a self-signed SSL on all of your services.
https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-debian-10<br>
set a self-signed SSL on all of your services<br>
<br>
TLS, or transport layer security, and its predecessor SSL, which stands for secure sockets layer, are web protocols used to wrap normal traffic in a protected, encrypted wrapper.<br>
Servers can send traffic safely between servers and clients without the possibility of messages being intercepted by outside parties. The certificate system also assists users in verifying the identity of the sites that they are connecting with.
### Step 1 — Creating the SSL Certificate for Apache<br>
TLS/SSL works by using a combination of a public certificate and a private key. The SSL key is kept secret on the server. It is used to encrypt content sent to clients. The SSL certificate is publicly shared with anyone requesting the content. It can be used to decrypt the content signed by the associated SSL key.<br>

#### Create self-signed key and certificate pair with OpenSSL:
```
$ sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt
```
#### Configure Apache tu use SSL:
We have created our key and certificate files under the /etc/ssl directory.
- Create configuration snippet to specify strong default SSL settings.
```
$ sudo vim /etc/apache2/conf-available/ssl-params.conf
```
add following:<br>
```
SSLCipherSuite EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH
SSLProtocol All -SSLv2 -SSLv3 
SSLHonorCipherOrder On
Header always set X-Frame-Options DENY
Header always set X-Content-Type-Options nosniff
# Requires Apache >= 2.4
SSLCompression off
SSLUseStapling on
SSLStaplingCache "shmcb:logs/stapling-cache(150000)"
# Requires Apache >= 2.4.11
SSLSessionTickets Off
```
- Modify the included SSL Apache Virtual Host (Default) file to point to our generated SSL certificates.
```
$ sudo vim /etc/apache2/sites-available/default-ssl.conf
```
add following:<br>
```
<IfModule mod_ssl.c>
        <VirtualHost _default_:443>
                ServerAdmin root@localhost
                ServerName 192.168.1.74
                DocumentRoot /var/www/html

                ErrorLog ${APACHE_LOG_DIR}/error.log
                CustomLog ${APACHE_LOG_DIR}/access.log combined

                SSLEngine on

                SSLCertificateFile      /etc/ssl/certs/ssl-cert-snakeoil.pem
                SSLCertificateKeyFile /etc/ssl/private/ssl-cert-snakeoil.key

                <FilesMatch "\.(cgi|shtml|phtml|php)$">
                                SSLOptions +StdEnvVars
                </FilesMatch>
                <Directory /usr/lib/cgi-bin>
                                SSLOptions +StdEnvVars
                </Directory>

        </VirtualHost>
</IfModule>
```
- Modify the HTTP Host File to Redirect to HTTPS:
```
$ sudo vim /etc/apache2/sites-available/000-default.conf
```
add following:<br>
```
Redirect "/" "https://192.168.1.74/"
```

