#Procedure to enable SSL for a Local Debian 10/11 Apache Web Server
All the instructions I found assume you have a domain or have a large network where DNS is running. While I could install DNS using bind9 and I have done so on other local networks; These instructions are just to allow you to use SSL (https://\<some local ip address\>).

Only by installing DNS through bind9 can you do https://\<some host name\>. That is *NOT* covered here.


The concept of enabling SSL is to have a Root Cert Authority bless your Apache web servers certificate. Normally the world only has a limited number of trusted root certificates already installed on everyones device. Instead you will become your own Root CA Authority, installing the Root Certificate on all your devices.

The instructions below cover how to do all this, using just the IP Address of your web server. No hostname or DNS required.


Everything here is pretty straight forward and can be found in other examples on the net.  I even quote a few and I thank them very much.  Sometimes it is just better to have a filled in example and one that works for a certain situation.  I hope this works for you.  If it doesn't, there are logs to look at that the system generates. Unfortunately I do not have the time to discuss any problems you may encounter.  I can tell you that I cut and pasted these exact lines so many times trying to understand it all, that what is here did work for me.

Take care,

John Talbot



#Step 1. Becoming Your own Certificate Authority (CA)
Kind of taken from:

  https://deliciousbrains.com/ssl-certificate-authority-for-local-https-development/ and other sources.

  Note: I wanted a designation that reflected services for my house. So SixtyFiveStableWay was born.  The files created are named for this but there is no hostname or FQDN, only an IP. Change the designation to your liking as well as the IP. At least this shows you a real example.

##Step 1A.  As usual, update.

```
   Shell> sudo apt-get update
```

##Step 1B.  Make sure OpenSSL is installed

```
   Shell> sudo apt-get install openssl
```

##Step 1C. Generate our Private Certificate Authority (CA) key.

   This will prompt for a passphase.  Keep this key and passphrase in
   a secure place.

   Note: A good CA passphrase is more than 30 characters and as much as 1834 characters.

```
      Shell> openssl genrsa -des3 -out 65StableWayCA.key 2048
```

##Step 1D. Create the root certificate.

This cert is good for 5 years (365\*5=1830) – you can change that.

```
   Shell> openssl req -x509 -new -nodes -key 65StableWayCA.key -sha256 -days 1830 -out 65StableWayRootCA.pem
```

Setup as follows for the root CA:

```
      Enter pass phrase for 65StableWayCA.key:  <Use above passhrase>
         (I saved this in 1Password)
      Country Name (2 letter code) [AU]:CA
      State or Province Name (full name) [Some-State]:Ontario
      Locality Name (eg, city) []:Kanata
      Organization Name (eg, company) [Internet Widgits Pty Ltd]:65 Stable Way
      Organizational Unit Name (eg, section) []:Root CA Admin
      Common Name (e.g. server FQDN or YOUR name) []:192.168.2.66
      Email Address []:SixtyFiveStableWay@gmail.com
```

#Step 2. Installing the Root Certificate on Various Devices

   When a root certificate is installed on a device, any certificates the root certificate blesses, will be valid.

##A. Installing the Root Certificate to the MacOS KeyChain

Step 1. Open the macOS Keychain app

Step 2. Select your private key file (i.e. SixtyFiveStableWayRootCA.pem)

Step 3. Select Keychains -> System and click on 192.168.2.66

Step 4. Expand the Trust section and Change the When using this
    certificate: select box to “Always Trust”

Step 5. Close the certificate window

Step 6. It will ask you to enter your password (or scan your finger), do that


##B. Installing root certificate on a IOS Device.

Step 1. Email the root certificate to yourself so you can access it on your iOS device

Step 2. Click on the SixtyFiveStableWayRootCA.crt attachment in the email on your iOS device

Step 3. Go to the settings app -> General -> Profile.

Step 4. Select '192.168.2.66' as a downloaded Profile.

Step 5. Click on Install on the top right.

Step 6A. This will show a page that says, "Unmanaged ROOT Certificate".

Step 6B. Click on Install on the top right.

Note: This certificat will not be trusted for websites until you enable it in Trust Settings.

Step 7. Go to Settings -> General  -> About -> Certificate Trust Settings

Step 8. Enable the Trust for Root Certificate 192.168.2.66

##C. Installing the Root Certificate on Debian

Step 1. Convert the .pem file to a .crt file

```
   Shell> openssl x509 -in ./65StableWayRootCA.pem -inform PEM -out ./65StableWayRootCA.crt
```

Step 2. Create the directory where certificates will be installed from

```
    Shell> sudo mkdir -p /usr/share/ca-certificates/extra
```

Step 3. Copy the .crt file to this directory

```
    Shell> sudo cp 65StableWayRootCA.crt /usr/share/ca-certificates/extra/
```

Note: /usr/share/ca-certificates and /usr/local/share/ca-certificates are essentially the same thing. One is controlled by a␣ config file the other is read automatically by the update-ca-certificates command used later.

Step 4. Append a line to /etc/ca-certificates.conf using \<folder name\>/\<.crt name\>:

```
    Shell> sudo echo "extra/65StableWayRootCA.crt" >> /etc/ca-certificates.conf
```

Step 5. Update certs non-interactively with sudo update-ca-certificates

```
    Shell> sudo update-ca-certificates
           ...
           Updating certificates in /etc/ssl/certs...
           1 added, 0 removed; done.
```

##D. Installing the Root Certificate to the Safari KeyChain

Step 1. Open Safari Menu (Tree lines on right)

Step 2. Select Privacy & Security

Step 3. Select View Certificates

Step 4. Select Import.

Step 5. Select the 65StableWayRootCA.pem file

Note: Your not done yet. You still need to do services like https (apache)

##Step 3. Installing Apache with SSL

Now that we’re a CA on all our devices, we can sign certificates for our Apache Web Server.

Step 3A. As usual, update first.

```
      Shell> sudo apt-get update
```

Step 3B.  Make sure Apache2 and OpenSSL is installed:

```
      Shell> sudo apt-get install apache2 openssl
```

Step 3C. Create a new directory for local certificates
   (-p means no error if existing, make parent directories as needed):

```
      Shell> sudo mkdir -p /etc/ssl/localcerts
```

Step 3D. First, we create a private key:

```
      Shell> openssl genrsa -out SixtyFiveStableWay.key 2048
```

Step 3E. Next we create a CSR. The name is a little different from above. They
   are not the same.
   The questions are similiar to above, but less important because what will be
   created will be blessed later by the root CA created above.

```
      Shell> openssl req -new -key SixtyFiveStableWay.key -out SixtyFiveStableWay.csr

      Country Name (2 letter code) [AU]:CA
      State or Province Name (full name) [Some-State]:Ontario
      Locality Name (eg, city) []:Kanata
      Organization Name (eg, company) [Internet Widgits Pty Ltd]:65 Stable Way
      Organizational Unit Name (eg, section) []:Household Admin
      Common Name (e.g. server FQDN or YOUR name) []:192.168.2.66
      Email Address []:SixtyFiveStableWay@gmail.com
      Please enter the following 'extra' attributes
      to be sent with your certificate request
      A challenge password []:
      An optional company name []:
```
      Note: The challenge password and optional company name is omitted so
            just hit return for them

Step 3F. Create the certificate used by the Apache web server

   You willl use our CSR, the CA private key, the CA certificate, and a config file that we need to create ourselves.

   The config file is needed to define the Subject Alternative Name (SAN)
   extension which is defined in this section (i.e. extension) of the
   certificate:

   I found this script that is what is more commonly used to generate
   the config file. What I did use is what is described as only needed
   and what I used to make it work.  You can glean some good information
   from the shell script though and even run it to see what the
   /tmp/openssl.cnf file it creates looks like.

   However, we will create a much simpler config file shown farther below.

```bash
#!/bin/bash
#using: OpenSSL 1.1.1c FIPS  28 May 2019 / CentOS Linux release 8.2.2004

C=US ; ST=Confusion ; L=Anywhere ; O=Private\ Subnet ; EMAIL=admin@company.com
BITS=2048
CN=RFC1918
DOM=company.com
SUBJ="/C=$C/ST=$ST/L=$L/O=$O/CN=$CN.$DOM"

openssl genrsa -out ip.key $BITS

SAN='\n[SAN]\nsubjectAltName=IP:192.168.1.0,IP:192.168.1.1,IP:192.168.1.2,IP:192.168.1.3,IP:192.168.1.4,IP:192.168.1.5,IP:192.168.1.6,IP:192.168.1.7,IP:192.168.1.8,IP:192.168.1.9,IP:192.168.1.10'\

cp /etc/ssl/openssl.cnf /tmp/openssl.cnf
echo -e "$SAN" >> /tmp/openssl.cnf


openssl req -subj "$SUBJ" -new -x509 -days 10950 \
    -key ip.key -out ip.crt -batch \
    -set_serial 168933982 \
    -config /tmp/openssl.cnf \
    -extensions SAN

openssl x509 -in ip.crt -noout -text

```

Step 3F. (Continued) Create SixtyFiveStableWay.ext config file with the following information:

```
# This is a comment line
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = IP:192.168.2.66

# Including this gave me an error "Loading extension section default"
# error: name=commonName, value=SixtyFiveStableWay
# Since it was a hostname, I did not want it anyway.
# commonName = SixtyFiveStableWay
```

Note: I tried just putting the IP address instead of the commonName but got the error message: SSL_ERROR_BAD_CERT_DOMAIN in any web browser I tried.

According to https://support.globalsign.com/ssl/general-ssl/securing-public-ip-address-ssl-certificates and https://cabforum.org/wp-content/uploads/CA-Browser-Forum-BR-1.6.3.pdf
An SSL certificate is typically issued to a Fully Qualified Domain Name (FQDN) such as "https://www.domain.com/". However, some organizations need an SSL certificate issued to a public IP address. This option allows you to specify a public IP address as the Common Name in your Certificate Signing Request (CSR). The issued certificate can then be used to secure connections directly with the public IP address.

So it can be possible to use an IP address. What that request looks like, I am not sure.  This however worked.

#Step 4. Now we put it all together to create the certificate

Step 4A. Create the final certificate

```
      Shell> openssl x509 -req -in SixtyFiveStableWay.csr -CA 65StableWayRootCA.pem -CAkey 65StableWayCA.key -CAcreateserial -out SixtyFiveStableWay.crt -days 825 -sha256 -extfile SixtyFiveStableWay.ext
```

   This will return with:

```
      Signature ok
      subject=C = CA, ST = Ontario, L = Kanata, O = 65 Stable Way, OU = Household Admin, CN = 192.168.2.66, emailAddress = SixtyFiveStableWay@gmail.com
      Getting CA Private Key
      Enter pass phrase for 65StableWayCA.key:
```

Note: I noticed the above command also created a file called SixtyFiveStableWay.srl.  This is a serial number file which is suposed to be used when you create other certificates.


Step 4B. Copy the SixtyFiveStableWay.crt file to /etc/ssl/localcerts/

```
      Shell> sudo cp SixtyFiveStableWay.crt /etc/ssl/localcerts/
```

Step 4C. Copy the SixtyFiveStableWay.key to /etc/ssl/private

```
      Shell> sudo cp SixtyFiveStableWay.key /etc/ssl/private/
```

Step 4D. Change ownership of these files.

```
      Shell> sudo chmod 600 /etc/ssl/localcerts/*.crt
      Shell> sudo chmod 600 /etc/ssl/private/*.key
```

Step 4E. Enable SSL:

```
      Shell> sudo a2ensite ssl
```

If you get "ERROR: Site ssl does not exist!" then do

```
      Shell> sudo a2ensite default-ssl
```

Step 4F. Now you need to edit the ssl configuration file in the /etc/apache2/sites-available directory.

```
    Shell> cd /etc/apache2/sites-available
    Shell> ls -l
```

For me, it looked like this:

```
      -rw-r--r-- 1 root root 1332 Apr  2  2019 000-default.conf
      -rw-r--r-- 1 root root 6338 Apr  2  2019 default-ssl.conf
```

Step 4G. Copy the default-ssl.conf to a new file named the same name as your IP name above – for this example:

```
       Shell> sudo cp default-ssl.conf 192.168.2.66.conf
```

Step 4H. The default-ssl.conf would have been enabled and since you
    are overriding it you need to disable it

```
       Shell> sudo a2dissite default-ssl.conf
```

Note: this removes the link from /etc/apache2/sites-available/default-ssl.conf to /etc/apache2/sites-enabled/default-ssl.conf

Step 4I. Edit SixtyFiveStableWay.conf

Step 4J. Change this line:

```
      <VirtualHost _default_:443>
     to this:

      <VirtualHost *:443>
```

and change these two lines:

```
      SSLCertificateFile    /etc$
      SSLCertificateKeyFile /etc$
   to
      SSLCertificateFile /etc/ssl/localcerts/SixtyFiveStableWay.crt
      SSLCertificateKeyFile /etc/ssl/private/SixtyFiveStableWay.key
```

Note: No matter how I tried to add "ServerName SixtyFiveStableWay" to the conf files in apache2/sites-available/\*.conf, I always got the daemon log of:

```
     apache2: Could not reliably determine the server's fully
     qualified domain name, using 127.0.1.1. Set the 'ServerName'
     directive globally to suppress this message.
```
I finally did what the message says and added: "ServerName 192.168.2.66" to the main apache2.conf file.

add

```
      ServerName 192.168.2.66
```

Step 4K. Save, close, then do:

```
      Shell> sudo a2ensite 192.168.2.66
```

Note: This just makes a link from the sites-available to the sites-enabled directory of /etc/apache2

#Step 5. Now restart Apache:

```
       Shell> sudo service apache2 restart
```


For Error: Unable to connect, Browser can't establish a connection to the server at 192.168.2.66

This error can be shown when either apache or virtual host itself cannot be reached over port 443.  You can check it with:

```
     Shell> netstat -ntpl | grep apache2
```

   In case apache does not listen on port 443 you should enable mod_ssl for apache:

```
     Shell> sudo a2enmod ssl
```

Seeing both ports enabled will show:

```
       tcp    0   0 0.0.0.0:443  0.0.0.0:*  LISTEN      4488/apache2
       tcp    0   0 0.0.0.0:80   0.0.0.0:*  LISTEN      4488/apache2
```

#Step 6. Let it Rip !!!

If you installed the root CA in all your devices you should then be able to go to https://192.168.2.66 and go directly to the Apache2 Debian Default Page WITHOUT ERRORS! Do not click though any untrusted web pages, except to maybe look at the certificate for errors. Never install it this way though. It defeats the  purpose.

# Footnotes

## Don't be afraid to look at /var/log/syslog

## Don't be afraid to look at /var/log/apache2/\*.log
