# Home NAS

Instruction how to setup TrueNas + NextCloud server with second replica TrueNas server

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
- Change `truenas_admin` login to `admin`

### Static IP
`Network`->`Interfaces`->`Edit` (on active connection):
- `DHCP`: `False`
- `Aliases` -> `Add`: `<local ip address>/24`
- Skip question about change Gateway

### Internet
`Network`->`Global Configuration`->`Settings`:
- `DNS Servers`
  - `Nameserver 1`: `<local router address>`


## NextCloud
### DataSets Structure
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
- (optional) Remove sceletron folder (cli only option)
- (optional) Disable Readme.md previews in folders

### Snapshots
- `DataSet`: `pool/nextcloud`
- `Recursive`: `True`
- `Snapshot Lifetime`: `2 WEEK` (default)
- `Naming Schema`: `nextcloud-%Y-%m-%d_%H-%M`
- `Schedule`: `once per day` (default)
- `Allow Taking Empty Snapshots`: `False`

### (Optional) Override trusted domains
- Using TrueNas Shell change NextCloud `config.php`:
```
cd /mnt/pool/nextcloud/app_data/config
nano config.php
```
- To `trusted_domains` add `<custom domain>`
- Change `overwriteprotocol` to `https`

### Nextcloud Office
- Install apps:
  - `Collabora Online - Built-in CODE Server`
  - `Nextcloud Office`
- `Administration settings` -> `Nextcloud Office`:
  - Chose: `Use the Built-in CODE - Collabora Online Development Edition`

## Nginx Proxy Manager
### DataSets Structure
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
dns_ovh_application_key = `<secret>`
dns_ovh_application_secret = `<secret>`
dns_ovh_consumer_key = `<secret>`
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
### DataSets Structure
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


## Resources
Postgres dataset: https://github.com/truenas/apps/issues/790  
Nextcloud office: https://youtu.be/sHU6XZ_b2hw?t=991