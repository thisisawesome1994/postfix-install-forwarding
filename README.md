# postfix-install-forwarding

</br>
  <b>1. Install Postfix<b>

```
sudo apt update
sudo apt-get install postfix

```
  </br>
  <b>2. Configure DNS Settings<b></br>
  </br>
You need to create an A record and MX record in your domain network settings:</br>
A Record = mail.yourdomain.com > Server IP > TTL 3600</br>
MX Record = yourdomain.com > mail.yourdomain.com > Priority 10 > TTL 14400</br>
</br>

  <b>3. Edit the Postfix Config File<b></br>
  </br>
  
```
sudo nano /etc/postfix/main.cf
```
Add the following lines below the line “alias_database = hash:/etc/aliases”</br>
```
virtual_alias_domains = localhost
virtual_alias_maps = hash:/etc/postfix/virtual
```
Add your domain to the “mydestination =” line.</br>
</br>
  <b>4. Add your Forwarding Email Addresses to the Virtual Alias File<b></br>
</br>
You will create the virtual postfix file using the nano command and add your email forwarding addresses.</br>
```
nano /etc/postfix/virtual
```
And edit as follow</br>
```
anything@yourdomain.com anything@forwardingemailaddress.com
```
To catch all emails sent to your domain, just use: </br>
```
@yourdomain.com anything@forwardingemailaddress.com

```
  <b>5. Open Port 25<b></br>
```
ufw allow 465
ufw allow 25
```
  
  <b>6. Change to port 465 smtps<b></br>

```
nano /etc/postfix/master.cf
```
    
  (add the following)</br>
  
```
465     inet  n       -       n       -       -       smtpd
  -o syslog_name=postfix/smtps
  -o smtpd_tls_wrappermode=yes
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_client_restrictions=permit_sasl_authenticated,reject

```
  <b>7. Map Alias Domain File and Restart Postfix<b></br>


```
sudo postmap /etc/postfix/virtual
sudo /etc/init.d/postfix reload
sudo service postfix restart 
 
```
  <b>8. Optional: Adding TLS security</br>
    - Create certificate inside /etc/postfix</b></br>
    
```
cd /etc/postfix
openssl req -nodes -newkey rsa:4096 -keyout postfix.key -out postfix.csr
```
  <b>9. Edit "/etc/postfix/main.cf" </b></br>
```
nano /etc/postfix/main.cf
```
    and add the following</br>
```
# TLS configuration starts here

tls_random_source = dev:/dev/urandom

# openssl_path=/usr/local/libressl/bin/openssl
# uncomment and edit the above if you're using a different "openssl" than the system's
# (in this case, LibreSSL)

# SMTP from your server to others
smtp_tls_key_file = /etc/postfix/postfix.key
smtp_tls_cert_file = /etc/postfix/postfix.crt
smtp_tls_CAfile = /etc/postfix/postfix.crt
smtp_tls_security_level = may
smtp_tls_note_starttls_offer = yes
smtp_tls_mandatory_protocols=!SSLv2,!SSLv3
smtp_tls_protocols=!SSLv2,!SSLv3
smtp_tls_loglevel = 1
smtp_tls_session_cache_database =
    btree:/var/lib/postfix/smtp_tls_session_cache

# SMTP from other servers to yours
smtpd_tls_key_file = /etc/postfix/postfix.key
smtpd_tls_cert_file = /etc/postfix/postfix.crt
smtpd_tls_CAfile = /etc/postfix/postfix.crt
smtpd_tls_security_level = may
smtpd_tls_auth_only = yes
smtpd_tls_mandatory_protocols=!SSLv2,!SSLv3
smtpd_tls_protocols=!SSLv2,!SSLv3
smtpd_tls_loglevel = 1
smtpd_tls_session_cache_database =
    btree:/var/lib/postfix/smtpd_tls_session_cache

# TLS configuration ends here
```
  <b>10. Edit /etc/postfix/master.cf</b></br>
```
nano /etc/postfix/master.cf
```
and add the following</br>
```
submission  inet  n     -       n       -       -       smtpd
    -o smtpd_etrn_restrictions=reject
    -o smtpd_sasl_auth_enable=yes
    -o smtpd_recipient_restrictions=permit_mynetworks,permit_sasl_authenticated,reject
    -o smtpd_client_restrictions=permit_mynetworks,permit_sasl_authenticated,reject
    -o smtpd_helo_restrictions=permit_mynetworks,permit
    -o smtpd_tls_security_level=encrypt
```
```
