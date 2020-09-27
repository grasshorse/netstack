# nginx-freenas

## HOW TO SET UP AN NGINX REVERSE PROXY WITH SSL TERMINATION IN FREENAS
A number of netstack services are externally available.  
A reverse proxy is used to correctly direct queries to the appropriate lan server. 

## JAIL CONFIGURATION
Run the reverse proxy in its own jail for isolation from other services. 
To do this, SSH into your FreeNAS host.

```
ssh root@192.168.128.5
```

## CREATE THE JAIL
Create the jail as follows:

```
iocage create -n reverse-proxy -r 11.2-RELEASE ip4_addr="vnet0|192.168.0.9/24" defaultrouter="192.168.0.1" vnet="on" allow_raw_sockets="1" boot="on"
```

Command explained:

| param | detail |
|-------|--------|
| iocage create | calls on the iocage command to create a new iocage jail |
| -n reverse-proxy | gives the jail the name ‘reverse-proxy’ |
| -r 11.2-RELEASE | specifies the release of FreeBSD to be installed in the jail |
| ip4_addr="vnet0-192.168.0.9/24" | the "-" should be a pipe: IP/mask for the jail, and the interface to use, vnet0 |
| defaultrouter="192.168.0.1" | specifies the router for your network |
| vnet="on" | enables the vnet interface |
| allow_raw_sockets="1" | enables raw sockets, which enables the use of traceroute and ping within the jail, as well as interactions with various network subsystems |
| boot="on" | enables the jail to be auto-started at boot time |

Detail can be found in the man page for iocage

See the status of the newly created jail, execute the following:
```
iocage list
```

This will present a print out similar to the following:
```
+-----+---------------+-------+--------------+----------------+
| JID |     NAME      | STATE |   RELEASE    |      IP4       |
+=====+===============+=======+==============+================+
| 1   | reverse-proxy | up    | 11.2-RELEASE | 192.168.0.9    |
+-----+---------------+-------+--------------+----------------+
```

Enter the jail by taking note of the JID value and executing the following:

```
jexec <JID> <SHELL>
```
For example,

```
jexec 1 tcsh
```

## INSTALL NGINX
Begin the installation process by updating the package manager, and installing nginx along with python:
```
pkg update
pkg install nginx python
```

Enable nginx so that the service begins when the jail is started
```
sysrc nginx_enable=yes
```

## SSL/TLS TERMINATION
Since the rest of this procedure involves making some decisions about whether or not to use SSL/TLS termination, we’ll discuss it here.
This guide is going to assume that the reverse proxy will be responsible for maintaining the certificates for all of the servers that it proxies to. 
This does not have to be the case, however. 

This configuration will use a wildcard certificate. 
That is, a certificate for the domain *.example.com, which is valid for all subdomains of example.com. 
This will simplify the process, as only one certificate needs to be obtained and renewed. 
However, one requirement of obtaining a wildcard certificate from LetsEncrypt is that a DNS-01 challenge is used to verify ownership for the domain. 
This means that HTTP-01 challenges cannot be used with this method, meaning that you must be using a DNS service that gives you control over your DNS records, or an API plugin to allow for DNS challenges. 
Certbot have published a list of supported DNS plugins that will enable you to perform a DNS challenge directly. 
If you’re using one of these providers, I recommend using these. 
Alternatively, if your DNS provider does not have a plugin, but you have access to edit the DNS records, you can manually configure a TXT record, as described in the certbot documentation. If neither of these alternatives are sufficient for you, acme.sh is a script that has perhaps wider compatability for a range of DNS Providers. Specific compatability is detailed in this community maintained list.

## CERTBOT INSTALLATION
Certbot is free, open source tool for obtaining and maintaining LetsEncrypt certificates. Install it as follows:
```
pkg install py37-certbot openssl
```

Additionally, you’ll need to install the appropriate plugin for DNS validation. To show a list of available plugins, execute:
```
pkg search certbot
```

At the time of writing, the (relevant) list of results looks like follows:
```
py37-certbot-1.0.0,1           Let's Encrypt client
py37-certbot-apache-1.0.0      Apache plugin for Certbot
py37-certbot-dns-cloudflare-1.0.0 Cloudflare DNS plugin for Certbot
py37-certbot-dns-cloudxns-1.0.0 CloudXNS DNS Authenticator plugin for Certbot
py37-certbot-dns-digitalocean-1.0.0 DigitalOcean DNS Authenticator plugin for Certbot
py37-certbot-dns-dnsimple-1.0.0 DNSimple DNS Authenticator plugin for Certbot
py37-certbot-dns-dnsmadeeasy-1.0.0 DNS Made Easy DNS Authenticator plugin for Certbot
py37-certbot-dns-gehirn-1.0.0  Gehirn Infrastructure Service DNS Authenticator plugin for Certbot
py37-certbot-dns-google-1.0.0  Google Cloud DNS Authenticator plugin for Certbot
py37-certbot-dns-linode-1.0.0  Linode DNS Authenticator plugin for Certbot
py37-certbot-dns-luadns-1.0.0  LuaDNS Authenticator plugin for Certbot
py37-certbot-dns-nsone-1.0.0   NS1 DNS Authenticator plugin for Certbot
py37-certbot-dns-ovh-1.0.0     OVH DNS Authenticator plugin for Certbot
py37-certbot-dns-rfc2136-1.0.0 RFC 2136 DNS Authenticator plugin for Certbot
py37-certbot-dns-route53-1.0.0 Route53 DNS Authenticator plugin for Certbot
py37-certbot-dns-sakuracloud-1.0.0 Sakura Cloud DNS Authenticator plugin for Certbot
py37-certbot-nginx-1.0.0       NGINX plugin for Certbot
```

Install the relevant plugin to you. For me, as mentioned, this is Route 53:
```
pkg install py37-certbot-dns-route53
```

## CONFIGURE DNS PLUGIN
To use the DNS plugin, you’re likely going to have to configure it. 
Consult the documentation for your relevant plugin. 
The py37-certbot-dns-route53 documentation lists the available methods to configure the Route 53 plugin, however Amazon have conveniently provided us with a CLI tool that will do it for us:
```
pkg install awscli
```

Before configuring it, you’ll need to create a Key Pair to provide, and limit, access to your AWS console. Bear in mind that if this server is compromised, the perpetrator will have access to this, so limiting the access this key pair has is advisable. The plugin documentation indicates that the following permissions are required:
```
route53:ListHostedZones
route53:GetChange
route53:ChangeResourceRecordSets
```

Initiate the configuration process:
```
aws configure
```

This will prompt you for four pieces of information:

AWS Access Key ID: From the key pair
AWS Secret Access Key: From the key pair
Default Region Name: The region closest to you, i.e. us-west-2. This should be available in your AWS dashboard
Default output format: text
Now, your configuration should be present in ~/.aws/config, and your credentials should be present in ~/.aws/credentials.

## REQUEST A WILDCARD CERTIFICATE
To obtain a certificate, simply execute the following command:
```
certbot certonly --dns-route53 -d '*.example.com'
```
This will undertake a DNS-01 challenge to verify access to the domain you substitute for example.com using the credentials in the plugin that you set up previously.

## CONFIGURE CERTIFICATE AUTO-RENEWAL
LetsEncrypt certificates are only valid for 90 days. 
To prevent these expiring, and having to manually repeat renew it, we can automate the renewal process. 
To do this, we’re going to add a cron job, which is essentially a command that runs at a specified interval. 
Set your default editor to nano and open up the crontab, where cron jobs are registered:
```
setenv EDITOR nano
crontab -e
```

Add the following line:
```
0 0,12 * * * /usr/local/bin/python -c 'import random; import time; time.sleep(random.random() * 3600)' && /usr/local/bin/certbot renew --quiet --deploy-hook "/usr/sbin/service nginx reload"
```

Save and Exit (Ctrl + X), and the cron job should be configured. This command will attempt to renew the certificate at midnight and noon every day.

One problem that I’ve had is that I’ve been able to get certificates to renew, however the certificate of the site still expires because the web server configuration hasn’t been reloaded. The --deploy-hook flag solves this issue for us, by reloading the web server when the certificate has been successfully updated.

Now we have our certificate to enable HTTPS, lets move on to configuring nginx.

## CONFIGURE NGINX
Before getting into specific configurations, it might be useful to outline the approach here. 
Because there is likely to be a number of duplications in the configuration files, some common snippets will be broken out into their own files to ease configuration management. 
The final list of configuration files we’ll end up with will be:
```
/usr/local/etc/nginx/nginx.conf
/usr/local/etc/nginx/vdomains/subdomain1.example.com.conf
/usr/local/etc/nginx/vdomains/subdomain2.example.com.conf
/usr/local/etc/nginx/snippets/example.com.cert.conf
/usr/local/etc/nginx/snippets/ssl-params.conf
/usr/local/etc/nginx/snippets/proxy-params.conf
/usr/local/etc/nginx/snippets/internal-access-rules.conf
```

## CERTIFICATE CONFIGURATION
To begin, we’ll start with the snippets:
```
cd /usr/local/etc/nginx
mkdir snippets
vi snippets/example.com.cert.conf
```

This file details the SSL/TLS certificate directives identifying the location of your certificates. Paste the following:
```
# certs sent to the client in SERVER HELLO are concatenated in ssl_certificate
ssl_certificate /usr/local/etc/letsencrypt/live/example.com/fullchain.pem;
ssl_certificate_key /usr/local/etc/letsencrypt/live/example.com/privkey.pem;

# verify chain of trust of OCSP response using Root CA and Intermediate certs
ssl_trusted_certificate /usr/local/etc/letsencrypt/live/example.com/chain.pem;
```
Remember to replace example.com with your domain, as requested when obtaining a wildcard certificate earlier. Save and Exit (Ctrl + X).

## SSL CONFIGURATION
Use the configuration generator at https://ssl-config.mozilla.org/ to generate a SSL configuration. 
I’d recommend only using either Intermediate or Modern. 
I’ve used Intermediate here because at the time of writing I had issues establishing a TLSv1.3 connection, whereas TLSv1.2 was consistently successful, however this compatability comes at the expense of security. The modern configuration is much more secure than the old configuration, for example.
```
vi snippets/ssl-params.conf
```
This is the contents of my file:
```
ssl_session_timeout 1d;
ssl_session_cache shared:MozSSL:10m;  # about 40000 sessions
ssl_session_tickets off;

# curl https://ssl-config.mozilla.org/ffdhe2048.txt > /usr/local/etc/ssl/dhparam.pem
ssl_dhparam /usr/local/etc/ssl/dhparam.pem;

# intermediate configuration
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
ssl_prefer_server_ciphers off;

# HSTS (ngx_http_headers_module is required) (63072000 seconds)
add_header Strict-Transport-Security "max-age=63072000" always;

# OCSP stapling
ssl_stapling on;
ssl_stapling_verify on;

# replace with the IP address of your resolver
resolver 192.168.0.1;
```
Replace the IP address of your resolver as directed, and then Save and Exit (Ctrl + X). If required by your desired configuration, you may also need to download the dhparam.pem certificate:
```
curl https://ssl-config.mozilla.org/ffdhe2048.txt > /usr/local/etc/ssl/dhparam.pem
```
Note that at the time of writing, the Modern configuration did not require this, but the Intermediate configuration did.

## PROXY HEADER CONFIGURATION
```
vi snippets/proxy-params.conf
```
Paste the following:
```
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_set_header X-Forwarded-Host $server_name;
proxy_set_header X-Forwarded-Ssl on;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
proxy_http_version 1.1;
```
Save and Exit (Ctrl + X)

## ACCESS POLICY CONFIGURATION
This is the policy that we’ll apply to services that you don’t want to be externally available, but still want to access it using HTTPS on your LAN.
```
vi snippets/internal-access-rules.conf
```

Populate it with the following:
```
allow 192.168.0.0/24;
deny all;
```

Replace the network with the subdomain relevant to you, and Save and Exit (Ctrl + X).

## VIRTUAL DOMAIN CONFIGURATION
Create a new directory for virtual domains:
```
mkdir vdomains
```
This directory will contain the configurations for each of the subdomains you wish to proxy to. You need to create one configuration file for each subdomain.

## EXTERNALLY AVAILABLE SUBDOMAIN
As an example, lets assume you have a Nextcloud server you want to proxy to such that it’s externally available outside your network. 
Create a configuration file for it:
```
vi vdomains/cloud.example.com.conf
```
Populate it as follows:
```
server {
        listen 443 ssl http2;

        server_name cloud.example.com;
        access_log /var/log/nginx/cloud.access.log;
        error_log /var/log/nginx/cloud.error.log;

        include snippets/example.com.cert.conf;
        include snippets/ssl-params.conf;

        location / {
                include snippets/proxy-params.conf;
                proxy_pass http://192.168.0.10;
        }
}
```
Then Save and Exit (Ctrl + X). Lets break this down so you understand what’s happening here. 
Each server can be handled within a server block. 
nginx iterates over the server blocks within it’s configuration in order until it finds one that matches the conditions of a request, and if no condition is matched, the server block marked as default_server is used.

The first statements:
```
listen 443 ssl http2;

server_name cloud.example.com;
access_log /var/log/nginx/cloud.access.log;
error_log /var/log/nginx/cloud.error.log;
```
This means that this server directive listens on port 443 for a HTTPS connection and enables HTTP/2 compatability. 
If a HTTPS request is made on port 443, and the Host header in the request matches the server_name directive, then this server block is matched and the directives are executed.

The access_log and error_log directives specify the location of these logs specifically for this server.
```
include snippets/example.com.cert.conf;
include snippets/ssl-params.conf;
```

These statement import the directives contained in the files we created earlier, specifically the certificate locations and the SSL parameters.
```
location / {
        include snippets/proxy-params.conf;
        proxy_pass http://192.168.0.10;
}
```

The location block is specific to the requested URI. In this case, the URI in question is /, the root. 
This means, that when the URL https://cloud.example.com is requested, this location directive is what’s executed. 
The include statement does the same thing as the snippets above; imports the directives contained in /usr/local/etc/nginx/snippets/proxy-params.conf that we created earlier. The proxy_pass statement is what redirects the request to the subdomain server. In this case, this is where the IP of the Nextcloud jail would go.

## INTERNALLY AVAILABLE SUBDOMAIN
If you don’t want this subdomain to be accessible outside of your local network, then you simply need to include the snippets/internal-access-rules.conf file we created earlier. Assuming you have a Heimdall server for example, your configuration file may be created as follows:

```
vi vdomains/heimdall.example.com.conf
```
And, assuming that the server is located at http://192.168.0.12, populate it as follows:
```
server {
        listen 443 ssl http2;

        server_name heimdall.example.com;
        access_log /var/log/nginx/heimdall.access.log;
        error_log /var/log/nginx/heimdall.error.log;

        include snippets/example.com.cert.conf;
        include snippets/ssl-params.conf;

        location / {
                include snippets/proxy-params.conf;
                include snippets/internal-access-rules.conf;
                proxy_pass http://192.168.0.12;
        }
}
```

### NGINX.CONF
Now, nginx only looks at /usr/local/etc/nginx/nginx.conf when inspecting configuration, so we have to tie everything we’ve just done in there. Open the file:

```
vi nginx.conf
```
The first thing you’ll need to do is disable the default configuration. You can do this by renaming it to nginx.conf.bak as follows:
```
mv nginx.conf nginx.conf.bak
```
Then create a new nginx.conf file for our new configuration:
```
vi nginx.conf
```
And populate it as follows:
```
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include mime.types;
    default_type application/octet-stream;
    sendfile on;
    keepalive_timeout 65;

    # Redirect all HTTP traffic to HTTPS
    server {
        listen 80 default_server;
        listen [::]:80 default_server;

        return 301 https://$host$request_uri;
    }

    # Import server blocks for all subdomains
    include "vdomains/*.conf";
}
```
Save and Exit (Ctrl + X). The important parts of this are the server block listening on port 80, and the include statement. The server block redirects all HTTP traffic to HTTPS to ensure that the SSL/TLS configuration we set up is being used, and the include statement imports the server blocks from all of the virtual domain configuration files. Now we need to start the service:
```
service nginx start
```
If it has already started, just reload it. This is the step you’ll have to take after all configuration changes:
```
service nginx reload
```

## ROUTER CONFIGURATION
Set up a NAT Port Forward to redirect all traffic received on port 80 at the WAN address to port 80 on the reverse proxy jail, and likewise for port 443. 
In pfSense (Firewall -> NAT), this looks like the following:

![portforward](./portforward.png)

DNS CONFIGURATION
In order to make these subdomains accessible both internally, and externally, you’ll need to add entries to a DNS resolver. 
To do this internally, you’ll need to add an entry for a Host Override, or whatever your router’s equivelant is. 
In pfSense, navigate to Service -> DNS Resolver -> Host Overrides. 
Assuming the subdomains proxy.example.com, cloud.example.com and heimdall.example.com, this would look like the following:

put in image

As can be seen, all subdomains are being resolved for the reverse proxy jail IP address of 192.168.0.9. 
For access to these services outside your network, you need to have a valid A record with your DNS provider. 
As an example, a valid A record would have the name cloud.example.com and the value would be your public IP address.

## CERTIFICATE AUTHORITY AUTHORIZATION (CAA) RECORDS
If you have a DNS provider that supports it, it might be a good idea to add a CAA Record. 
A CAA record essentially allows you to declare which certificate authorities you actually use, and forbids other certificate authorities from issueing certificates for your domain. 
You can read more about these at SSLMate. 
SSLMate also provide a configuration tool to help you auto-generate your CAA record configuration.

