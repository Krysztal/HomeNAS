# Home NAS
## BIOS
Fan configuratin:  
Advanced -> Hardware Monitor -> Smart Fan Function:  
  - Fan off temperature limit: 42
  - Fan start temperature limit: 45

Power Consumption configuration:  
Advanced -> Power & Performence -> CPU - Power Managment Control:
 - Boost performentce mode: Max Non-Turbo Performance

## TrueNAS
### Instalation
  - change `truenas_admin` login to `admin`
### Static IP
At `Network`->`Interfaces`->`Edit` (on active connection):
- DHCP: disable
- Aliases -> Add: `<local ip address>/24`

Skip question about change Gateway


### Internet
At `Network`->`Global Configuration`->`Settings`:
- DNS Servers:
  - Nameserver 1: `<local router address>`


## NextCloud
Datasets Structure:
- nextcloud (Generic + Compression Level: OFF)
  - app_data (App)
  - user_data (Generic)
  - postgres_data (Generic + 999:999 as owner)

App instalation:
- Host: public ip/domain
- Cron: enable
- GPU Configuration
  - Passthrough available (non-NVIDIA) GPUs: enable

NextCloud configuration:
- Disable apps
  - Versions
  - Weather status
- (optional) Remove sceletron folder (cli only option)
- (optional) Disable Readme.md preview in folders

Snapshots:
- DataSet: pool/nextcloud
- Recursive: enable
- Snapshot Lifetime: default (2 WEEK)
- Naming Schema: nextcloud-%Y-%m-%d_%H-%M
- Schedule: default (once per day)
- Allow Taking Empty Snapshots: disable

### Override trusted domains (Optional):
Go to TrueNas Shell and configure nexcloud config.php  
```
cd /mnt/pool/nextcloud/app_data/config
nano config.php
```
- add to 'trusted_domains' =>  '`<custom domain>`'
- change 'overwriteprotocol' => 'https'

### Nextcloud Office
Install apps:
- Collabora Online - Built-in CODE Server
- Nextcloud Office

On Administration settings:  
- Nextcloud Office:
  - chose: Use the Built-in CODE - Collabora Online Development Edition

## Nginx Proxy Manager
Datasets Structure:
- nginx_proxy_manager (Generic + Compression Level: OFF)
  - certs_storage (App)
  - data_storage (App)

OVH configuration:
- Buy domain on [OVH](https://www.ovh.com/)

Nginx configuration:
- Change login:
  - Email: admin@example.com
  - Password: changeme
- SSL Certificates add for each domain:
  - Domain Names: `<domain from ovh>`
  - Email Address for Let's Encrypt: your email
  - Use a DNS Challenge: `True`
  - DNS Provider: OVH
  - Credentials File Content: `<result of OVH token generation>`
- Proxy Hosts add for each domain:
  - Details:
    - Domain Names: `<domain from ovh>`
    - Scheme: http
    - Forward Hostname / IP: `<application IP>`
    - Forward Port: `<application port>`
    - Cache Assets: `True`
    - Block Common Exploits: `True`
    - Websockets Support: `True`
    - Access List: (default) Publicaly Accessible
  - SSL:
    - SSL Certificate:  `<created certificate>`
    - Force SSL: `True`
    - HSTS Enabled: `True`

### OVH Token Generation
Url: https://www.ovh.com/auth/api/createToken

Application name: nginx-proxy-manager  
Application description: Nginx Proxy Manager

Rights:  
Add:
  - GET /domain/zone/*
  - POST /domain/zone/*
  - DELETE /domain/zone/*

Result:
```
dns_ovh_endpoint = ovh-eu
dns_ovh_application_key = `<secret>`
dns_ovh_application_secret = `<secret>`
dns_ovh_consumer_key = `<secret>`
```

## Jellyfin
Datasets Structure:
  - jellyfin (Generic)
    - cache_storage (App)
    - config_storage (App)
    - transcode_storage (Generic)
    - library_storage (Generic)

App instalation:
- Published Server URL: public ip/domain
- Additional Storage. Mount Path: /library
- GPU Configuration
  - Passthrough available (non-NVIDIA) GPUs: enable


## qBittorrent
App instalation:
Password setup:
  - add to qBittorrent.config line to Preferences section: WebUI\Password_PBKDF2="@ByteArray(ARQ77eY1NUZaQsuDHbIMCA==:0WMRkYTUWVT9wVvdDtHAjU9b3b7uB8NR1Gur2hmQCvCDpm39Q+PsJRJPaCU51dEiz+dTzh8qbPsL8WkFljQYFQ==)"
  - default username: admin, password: adminadmin
Auth setup:
  - Tool -> Options -> WebUI -> Authentication:
    - chnage login and password
    - Bypass authentication for clients in whitelisted IP subnets: `<local ip address>/24`


## Links:
Postgres dataset: https://github.com/truenas/apps/issues/790  
Nextcloud office: (ru) https://youtu.be/sHU6XZ_b2hw?t=991