## You have to set a web server who should BE available on the VM’s IP or an host (init.login.com for exemple). About the packages of your web server, you can choose between Nginx and Apache. You have to set a self-signed SSL on all of your services.
https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-debian-10
TLS, or transport layer security, and its predecessor SSL, which stands for secure sockets layer, are web protocols used to wrap normal traffic in a protected, encrypted wrapper.<br>
Servers can send traffic safely between servers and clients without the possibility of messages being intercepted by outside parties. The certificate system also assists users in verifying the identity of the sites that they are connecting with.
### Step 1 — Creating the SSL Certificate for Apache<br>
TLS/SSL works by using a combination of a public certificate and a private key. The SSL key is kept secret on the server. It is used to encrypt content sent to clients. The SSL certificate is publicly shared with anyone requesting the content. It can be used to decrypt the content signed by the associated SSL key.<br>

#### Create self-signed key and certificate pair with OpenSSL:
openssl: This is the basic command line tool for creating and managing OpenSSL certificates, keys, and other files.
```
$ sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt
```


