### smtp_server_docker_mailserver
How to Setup Docker Mailserver on AWS EC2 And Deploy Ubuntu 2022
Docker Mailserver is free and open source.

### Requirements:
1. Request AWS Support Open Port 25 for smtp server.
2. Setup on aws RDNS ( Reverse Domain Name System ).
3. A registered domain name with access to DNS settings.
4. t2 or t3 medium ec2 2Gb Ram or 4Core
5. Static IP ( to associate ec2 )
6. Install Docker Latest Version
7. Create subdomain mail ( mail.example.com ) In DNS
8. Nginx On host machine
9. Install ssl ( USE Letsencrypt Use )

### Enable Parameters In mailserver.env
1. POSTMASTER_ADDRESS=postmaster@example.com
2. ENABLE_UPDATE_CHECK=1
3. UPDATE_CHECK_INTERVAL=1d
4. ENABLE_OPENDKIM=1
5. ENABLE_OPENDMARC=1
6. ENABLE_POLICYD_SPF=1
7. ENABLE_POP3=1
8. ENABLE_IMAP=1
9. ENABLE_CLAMAV=1
10. ENABLE_MTA_STS=1
11. RELAY_PORT=25

### DNS Record Setup
To add this to a GitHub description in a clear and understandable way, you can format it as follows:
DNS Records

To configure DNS settings for this server, use the following records:
@, mail, txt, MX, Dmarc, DKIM, SPF
### important Note
Replace example.com

|   Host   |   Record Type   |      Value           | Priority |
|----------|-----------------|----------------------|----------|
| `@`      | A               | *Server IP*          | -        |
| `mail`   | A               | *Server IP*          | -        |
| `@`      | MX              | `mail.example.com`   | 10       |
| `mail`   | MX              | `v=DMARC1; p=quarantine; adkim=r; aspf=r; pct=100`  | 10       |
| `_dmarc` | txt             | `v=spf1 include:mail.example.com -all`              | -        |

- **Host**: `@` refers to the root domain (e.g., `example.com`).
- **Record Type**:
  - **A Record**: Maps a domain to an IP address.
  - **MX Record**: Directs mail to a specified mail server.
- **Priority**: Lower values indicate higher priority.


We need to edit **compose.yaml** file to set our hostname. Use your favorite editor.
Replace example.com
```bash
hostname: mail.example.com
```

### Deploy ON Docker Container
To write the command in Markdown so it’s easy to copy on GitHub, use triple backticks with `bash` for syntax highlighting, like this:

```bash
docker compose up -d
```
### Create an Email Address From Mailserver Container
Replace example.com
```bash
docker exec -ti mailserver setup email add info@example.com
```
### Then run the following command to get the DKIM key.
```bash
docker exec -ti mailserver setup config dkim
```
### Find the docker-data Folder
```bash
cd docker-data/dms/config/opendkim/keys/example.com/
```
```bash
cat mail.txt
```
![DKIM-Docker-Mailserver](https://github.com/user-attachments/assets/89afbbab-8730-4bdd-80d5-e59e7fb18679)

Now we have the DKIM key and we will add this to our DNS records. ### Do not include parenthesis and quotation marks. ###

### Add This Key KDIM in DNS Record
|           Host             |   Record Type   |      Value   | Priority |
|----------------------------|-----------------|--------------|----------|
| `mail._domainkey`          | txt             | add dkim key | -        |

### For examples add key
```bash
v=DKIM1; h=sha256; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAwf32ZQtSMObL/jRq9RN+A5jrYsbXkIZnEdOY3RW5wFgH+G8rN/Lcu8iCkHpp9nt0xBEG6Aksq76wLDa2hPgFKoRAYZmCIrFInhsVgBgTxk2gAmauW4rZExevM3FZE1TzeMsfQHB78AJMNiXKdQpRCR+ivOvxH9ahx9TucW+Nc+03zYyfDB5I12fh6/hYnN0MF4xaDuu7Ddgrjeh/eukYYQOUEtxPOm21BPVCiHFhdGX3Nk08rRr1ZZN8807hsJZj4+aCStmk4We+ik/R/x8noa0r2rHVAc2iNO5kklmt/34ueMd+ZPmZw3DaGvu9KRuXuBjcnX9B/xXCUfJQqeuM5QIDAQAB
```
### Install Roundcube ( access WEBUI mailserver from roundcube ) 
Important node: open compose file and replace ( example.com )
```bash
docker compose up -d
```

### Install Nginx latest version 
```bash
sudo apt-get install nginx -y
```
Go to this directory
```bash
cd /etc/nginx/sites-available/
```
create host
```bash
sudo nano file-name
```

### Paste this file ( Virtual Host ) and Replace example.com and Replace ec2-static-public-ip
```bash
server {
    server_name mail.example.com www.mail.example.com;

    # Gzip compression
    gzip on;
    gzip_comp_level 6;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml text/javascript image/svg+xml;

    location / {
        proxy_pass http://ec2-static-public-ip-:8080; # Docker container running Roundcube
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Static files configuration
    location /web/static/ {
        proxy_pass http://ec2-static-public-ip:8080;
        proxy_cache_valid 200 90m;
        proxy_buffering on;
        expires 864000;
        add_header Cache-Control "public";
    }

    listen 443 ssl; # Enable SSL
    ssl_certificate /etc/letsencrypt/live/mail.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/mail.example.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
}

server {
    if ($host = mail.example.com) {
        return 301 https://$host$request_uri;
    }

    listen 80;
    server_name mail.example.com www.mail.example.com;
    return 404;
}
```
save and clone this file 

### Install ssl ( Letsencrypt ) or Follow this article
https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-22-04
Step 1 — Installing Certbot
```bash
sudo apt install certbot python3-certbot-nginx
sudo nginx -t
sudo systemctl reload nginx
```

Step 2 — Obtaining an SSL Certificate
```bash
sudo certbot --nginx -d example.com
```




