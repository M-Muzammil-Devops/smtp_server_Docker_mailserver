### smtp_server_Docker_mailserver
How to Setup Docker Mailserver on AWS EC2 And Deploy Ubuntu 2022
Docker Mailserver is free and open source.
Prerequisites:
1. Request AWS Support Open Port 25 for smtp server.
2. Setup on aws RDNS ( Reverse Domain Name System ).
3. A registered domain name with access to DNS settings.
4. t2 or t3 medium ec2 2Gb Ram or 4Core

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

|   Host   |   Record Type   |      Value           | Priority |
|----------|-----------------|----------------------|----------|
| `@`      | A               | *Server IP*          | -        |
| `mail`   | A               | *Server IP*          | -        |
| `@`      | MX              | `mail.example.com`   | 10       |

- **Host**: `@` refers to the root domain (e.g., `example.com`).
- **Record Type**:
  - **A Record**: Maps a domain to an IP address.
  - **MX Record**: Directs mail to a specified mail server.
- **Priority**: Lower values indicate higher priority.
