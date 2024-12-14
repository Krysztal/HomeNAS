# Home NAS
Instruction how to setup TrueNas + NextCloud server with second replica TrueNas server.

## Table of contents
- [BIOS](#bios)
  - [Fan configuration](#fan-configuration)
  - [Power Consumption configuration](#power-consumption-configuration)
- [TrueNAS](#truenas)
  - [Instalation](#instalation)
  - [Static IP](#static-ip)
  - [Internet](#internet)
  - [Data protection](#data-protection)
- [NextCloud](#nextcloud)
  - [DataSets structure](#datasets-structure)
  - [App instalation](#app-instalation)
  - [NextCloud configuration](#nextcloud-configuration)
  - [Snapshots](#snapshots)
  - [(Optional) Override trusted domains](#optional-override-trusted-domains)
  - [Nextcloud Office](#nextcloud-office)
- [Nginx Proxy Manager](#nginx-proxy-manager)
  - [DataSets structure](#datasets-structure-1)
  - [OVH configuration](#ovh-configuration)
  - [OVH Token Creation](#ovh-token-creation)
  - [Nginx configuration](#nginx-configuration)
- [Jellyfin](#jellyfin)
  - [DataSets structure](#datasets-structure-2)
  - [App instalation](#app-instalation-1)
- [qBittorrent](#qbittorrent)
  - [App instalation](#app-instalation-2)
- [References](#references)


## BIOS
### Fan configuration
`Advanced` -> `Hardware Montor` -> `Smart Fan Function`:  
- `Fan off temperature limit`: `42`
- `Fan start temperature limit`: `45`

### Power Consumption configuration
`Advanced` -> `Power & Performence` -> `CPU - Power Managment Control`:
- `Boost performentce mode`: `Max Non-Turbo Performance`


## TrueNAS
### Instalation
- Change login `truenas_admin` to `admin`

### Static IP
`Network`->`Interfaces`->`Edit` (on active connection):
- `DHCP`: `False`
- `Aliases` -> `Add`: `<local ip address>/24`
- Skip question about change Gateway

### Internet
`Network`->`Global Configuration`->`Settings`:
- `DNS Servers`
  - `Nameserver 1`: `<local router address>`

### Data protection
- Scrub:
  - `Pool`: `pool`
  - `Threshold Days`: `30`
  - `Description`: `Every 4 month`
  - `Schedule`: `Custom (0 2 7 2,6,10 *)`
    - `Minutes`: `0`
    - `Hours`: `2`
    - `Days of Month`: `7`
    - `Days of Week`: `all`
    - `Months`: `2,6,10`
  - `Enabled`: `True`
- Long SMART:
  - `All Disks`: `True`
  - `Type`: `LONG`
  - `Description`: `Every 2 month`
  - `Schedule`: `Custom (0 2 7 1,3,5,7,9,11 *)`
    - `Hours`: `2`
    - `Days of Month`: `7`
    - `Days of Week`: `all`
    - `Months`: `1,3,5,7,9,11`
- Short SMART:
  - `All Disks`: `True`
  - `Type`: `SHORT`
  - `Description`: `Twice per month`
  - `Schedule`: `Custom (0 2 1,15 * *)`
    - `Hours`: `2`
    - `Days of Month`: `1,15`
    - `Days of Week`: `all`
    - `Months`: `all`


## NextCloud
### DataSets structure
- `nextcloud` (`Generic` + `Compression Level`: `OFF`)
  - `app_data` (`App`)
  - `user_data` (`Generic`)
  - `postgres_data` (`Generic` + `999:999` as owner)

### App instalation
- `Host`: `<public ip/domain>`
- `Cron`: `True`
- `GPU Configuration`
  - `Passthrough available (non-NVIDIA) GPUs`: `True`

### NextCloud configuration
- Disable apps:
  - `Versions`
  - `Weather status`
- `Administration settings` -> `Basic settings` -> `Background jobs`:
  - Chose: `Cron (Recommended)`
- (optional) Remove sceletron folder (cli only option)
- (optional) Disable Readme.md previews in folders

### Snapshots
- `DataSet`: `pool/nextcloud`
- `Recursive`: `True`
- `Snapshot Lifetime`: `2 WEEK` (default)
- `Naming Schema`: `nextcloud-%Y-%m-%d_%H-%M`
- `Schedule`: `Daily (0 0 * * *)` (default: once per day)
- `Allow Taking Empty Snapshots`: `False`

### (Optional) Override trusted domains
- Using TrueNas Shell change NextCloud `config.php`:
```
cd /mnt/pool/nextcloud/app_data/config
nano config.php
```
- Add `<custom domain>` to `trusted_domains`
- Change to `https` at `overwriteprotocol`

### Nextcloud Office
- Install apps:
  - `Collabora Online - Built-in CODE Server`
  - `Nextcloud Office`
- `Administration settings` -> `Nextcloud Office`:
  - Chose: `Use the Built-in CODE - Collabora Online Development Edition`

## Nginx Proxy Manager
### DataSets structure
- `nginx_proxy_manager` (`Generic` + `Compression Level`: `OFF`)
  - `certs_storage` (`App`)
  - `data_storage` (`App`)

### OVH configuration
- Buy domain at [OVH](https://www.ovh.com/)

### OVH Token Creation
Using Url: [OVH Create Token](https://www.ovh.com/auth/api/createToken):
- `Application name`: `nginx-proxy-manager`
- `Application description`: `Nginx Proxy Manager`
- `Rights`:
  - `GET` `/domain/zone/*`
  - `POST` `/domain/zone/*`
  - `DELETE` `/domain/zone/*`

Create result file:
```
dns_ovh_endpoint = ovh-eu
dns_ovh_application_key = <secret>
dns_ovh_application_secret = <secret>
dns_ovh_consumer_key = <secret>
```

### Nginx configuration
- Change login from:
  - `Email`: `admin@example.com`
  - `Password`: `changeme`
- Create `SSL Certificates` for each domain:
  - `Domain Names`: `<domain from OVH>`
  - `Email Address for Let's Encrypt`: `<your email>`
  - `Use a DNS Challenge`: `True`
  - `DNS Provider`: `OVH`
  - `Credentials File Content`: `<OVH Token Creation result file>`
- Create `Proxy Hosts` for each domain:
  - `Details`:
    - `Domain Names`: `<domain from OVH>`
    - `Scheme`: `http`
    - `Forward Hostname / IP`: `<application IP>`
    - `Forward Port`: `<application port>`
    - `Cache Assets`: `True`
    - `Block Common Exploits`: `True`
    - `Websockets Support`: `True`
    - `Access List`: `Publicaly Accessible` (default)
  - `SSL`:
    - `SSL Certificate`: `<created certificate>`
    - `Force SSL`: `True`
    - `HSTS Enabled`: `True`


## Jellyfin
### DataSets structure
  - `jellyfin` (`Generic`)
    - `cache_storage` (`App`)
    - `config_storage` (`App`)
    - `transcode_storage` (`Generic`)
    - `library_storage` (`Generic`)

### App instalation
- `Published Server URL`: `<public ip/domain>`
- `Additional Storage. Mount Path`: `/library`
- `GPU Configuration`
  - `Passthrough available (non-NVIDIA) GPUs`: `True`


## qBittorrent
### App instalation
- Password setup:
  - At `qBittorrent.config` add line to `Preferences section`: `WebUI\Password_PBKDF2="@ByteArray(ARQ77eY1NUZaQsuDHbIMCA==:0WMRkYTUWVT9wVvdDtHAjU9b3b7uB8NR1Gur2hmQCvCDpm39Q+PsJRJPaCU51dEiz+dTzh8qbPsL8WkFljQYFQ==)"`
  - `default username`: `admin`
  - `password`: `adminadmin`
- Auth setup:
  - `Tool` -> `Options` -> `WebUI` -> `Authentication`:
    - Chnage login and password
    - `Bypass authentication for clients in whitelisted IP subnets`: `<local ip address>/24`


## References
Postgres DataSet: https://github.com/truenas/apps/issues/790  
NextCloud Office: https://youtu.be/sHU6XZ_b2hw?t=991  
Data protection: https://www.youtube.com/watch?v=dP0wagQVctc