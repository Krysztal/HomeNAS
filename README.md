# Home NAS
Instruction how to setup TrueNas + NextCloud main server with second backup TrueNas server.

## Table of contents
- [Home NAS](#home-nas)
  - [Table of contents](#table-of-contents)
  - [BIOS](#bios)
    - [Fan configuration](#fan-configuration)
    - [Power Consumption configuration](#power-consumption-configuration)
  - [OVH](#ovh)
    - [Configuration](#configuration)
    - [Token creation](#token-creation)
  - [TrueNAS](#truenas)
    - [Timezone](#timezone)
    - [Static IP](#static-ip)
    - [Internet](#internet)
    - [Email](#email)
    - [Certificates](#certificates)
    - [SSH (only main server)](#ssh-only-main-server)
    - [Pool](#pool)
    - [Data protection](#data-protection)
  - [NextCloud](#nextcloud)
    - [DataSets structure](#datasets-structure)
    - [App instalation](#app-instalation)
    - [Change config file](#change-config-file)
    - [NextCloud configuration](#nextcloud-configuration)
    - [Nextcloud Office](#nextcloud-office)
    - [Snapshots](#snapshots)
  - [DDNS Updater](#ddns-updater)
    - [DataSets structure](#datasets-structure-1)
    - [App instalation](#app-instalation-1)
    - [OVH configuration](#ovh-configuration)
  - [NextCloud client (optional)](#nextcloud-client-optional)
    - [Android](#android)
  - [Jellyfin (optional)](#jellyfin-optional)
    - [DataSets structure](#datasets-structure-2)
    - [App instalation](#app-instalation-2)
  - [qBittorrent (optional)](#qbittorrent-optional)
    - [Configuration](#configuration-1)
  - [Nginx Proxy Manager (deprecated)](#nginx-proxy-manager-deprecated)
    - [DataSets structure](#datasets-structure-3)
    - [Nginx configuration](#nginx-configuration)
  - [References](#references)


## BIOS
### Fan configuration
`Advanced` -> `Hardware Montor` -> `Smart Fan Function`:  
- `Fan off temperature limit`: `42`
- `Fan start temperature limit`: `45`

### Power Consumption configuration
`Advanced` -> `Power & Performence` -> `CPU - Power Managment Control`:
- `Boost performentce mode`: `Max Non-Turbo Performance`


## OVH
### Configuration
Buy domain at [OVH](https://www.ovh.com/)

### Token creation
Using Url: [OVH Create Token](https://www.ovh.com/auth/api/createToken):
- `Application name`: `CRT`
- `Application description`: `CRT`
- `Validity`: `Unlimited`
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


## TrueNAS
### Timezone
`System` -> `General Settings` -> `Localization`:
- `Timezone`: `<timezone>`

### Static IP
`Network`->`Interfaces`->`Edit` (on active connection):
- `DHCP`: `False`
- `Aliases` -> `Add`: `<local ip address>/24`
- Skip question about change Gateway

### Internet
`Network`->`Global Configuration`->`Settings`:
- `DNS Servers`
  - `Nameserver 1`: `<local router address>`
  - `IPv4 Default Gateway`: `<local router address>`

### Email
- `System` -> `Alert Settings` -> `Add` or edit existing:
  - `Name`: `E-Mail`
  - `Enabled`: `True` (default)
  - `Type`: `E-Mail`
  - `Level`: `Warning` (default)
  - `Override Admin Email`: `<emails separete by ,(comma)>`
- `System` -> `General Settings` -> `Email` -> `Settings`:
  - `Send Mail Method`: `<mail method>`
  - `From Email`: `<existing email>`
  - `From Name`: `<server name>`

### Certificates
- `Credentials` -> `Certificates`:
  - `ACME DNS-Authenticators` -> `Add`:
    - `Name`: `OVH`
    - `Authenticator`: `OVH`
    - `Application Key`: `<from OVH token>`
    - `Application Secret`: `<from OVH token>`
    - `Consumer Key`: `<from OVH token>`
    - `Endpoint`: `<from OVH token>`
  - `Certificate Signing Requests` -> `Add`:
    - `Identifier and Type`:
      - `Name`: `CSR`
      - `Type`: `Certificate Signing Request`
      - `Profile`: `HTTPS RSA Certificate`
    - `Certificate Options`: (default)
    - `Certificate Subject`:
      - `Country`: `<your country>`
      - `State`: `--`
      - `Locality`: `--`
      - `Organization`: `--`
      - `Email`: `<your email>`
      - `Subject Alternative Name`: `<sub domain>` or `<domain from OVH>`
    - `Extra Constraints`: (default)
    - `Save`
    - Click `Create ACME Certificate`:
      - `Identifier`: `CRT`
      - `Terms of Service`: `True`
      - `Renew Certificate Days`: `30`
      - `ACME Server Directory URI`: `Let's Encrypt Production Directory`
      - `Domains`: `OVH`
- `System` -> `General Settings` -> `GUI` -> `Settings`:
  - `GUI SSL Certificate`: `CRT`

### SSH (only main server)
`System` -> `Services` -> `SSH`:
- `Running`: `True`
- `Start Automatically`: `True`

### Pool
`Storage` -> `Create pool`:
- `General Info`
  - `Name`: `pool`
  - `Encryption`: `False`
- `Data`
  - `Layout`: `Stripe` (for one disk)
- Skip `Log`, `Spare`, `Cache`, `Metadata`, `Dedup`

`Datasets` -> `pool` -> `Dataset Details` -> `Edit` -> `Advanced Options`:
- `Compression Level`: `OFF`

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
- Replication (only backup server):
  - `Source Location`: `On a Different System`
  - `SSH Connection`: `Add New`
    - `Name`: `<main server name>`
    - `Setup Method`: `Semi-automatic (TrueNAS only)`
    - `TrueNAS URL`: `<main server url>`
    - `Admin Username`: `truenas_admin`
    - `Admin Password`: `<password>`
    - `Username`: `root`
    - `Private Key`: `Generate New`
  - `Source`: `pool/nextcloud`
  - `Destination`: `pool/nextcloud`
  - `Recursive`: `True`
  - `Include snapshots with the name`: `Naming Schema`
  - `Naming Schema`: `auto-%Y-%m-%d_%H-%M`
  - `SSH Transfer Security`: `Encryption (more secure, but slower)`
  - `Use Sudo For ZFS Commands`: `True`
  - `Task Name`: `pool/nextcloud - pool/nextcloud`
  - `Replication Schedule`: `Custom (30 0 * * *)`
    - `Minutes`: `30`
    - `Hours`: `0`
    - `Days of Month`: `*`
    - `Days of Week`: `all`
    - `Months`: `all`
  - `Destination Snapshot Lifetime`: `Custom`
    - `2` `Weeks`


## NextCloud
### DataSets structure
- `nextcloud` (`Generic` + `Compression Level`: `OFF`)
  - `app_data` (`App`)
  - `user_data` (`Generic`)
  - `postgres_data` (`Generic` + `999:999 (netdata:docker)` as owner)

### App instalation
- `Postgres Image (CAUTION)`: `Postgres 17`
- `APT Packages`: `ffmpeg`
- `Host`: `<public ip/domain without https://>`
- `Cron`: `True`
- `GPU Configuration`
  - `Passthrough available (non-NVIDIA) GPUs`: `True`

### Change config file
- Using TrueNas Shell change NextCloud `config.php`:
```
nano /mnt/pool/nextcloud/app_data/config/config.php
```
- Change `overwrite.cli.url` to `https://localhost`
- Add `'overwriteprotocol' => 'https'`
- Add `'overwritehost' => '<domain from OVH>'`,
- Add `'maintenance_window_start' => 0` (in UTC 1:00 or 2:00 for PL)
- Add `<domain from OVH>` to `trusted_domains` (if not exist)

### NextCloud configuration
- Disable apps:
  - `Versions`
  - `Weather status`
- Install apps:
  - `End-to-End Encryption`
- `Administration settings` -> `Basic settings` -> `Background jobs`:
  - Chose: `Cron (Recommended)`
- (optional) Remove sceletron folder (cli only option)
- (optional) Disable Readme.md previews in folders

### Nextcloud Office
- Install apps:
  - `Collabora Online - Built-in CODE Server`
  - `Nextcloud Office`
- `Administration settings` -> `Nextcloud Office`:
  - Chose: `Use the Built-in CODE - Collabora Online Development Edition`

### Snapshots
- `DataSet`: `pool/nextcloud`
- `Recursive`: `True`
- `Snapshot Lifetime`: `2 WEEK` (default)
- `Naming Schema`: `auto-%Y-%m-%d_%H-%M` (default)
- `Schedule`: `Daily (0 0 * * *)` (default: once per day)
- `Allow Taking Empty Snapshots`: `True` (default)


## DDNS Updater
### DataSets structure
  - `ddns_updater` (`App`)

### App instalation
  - `Config`:
    - `Provider`: `OVH`
    - `Domain`: `<sub domain>` or `<domain from OVH>` 
    - `Mode`: `Dynamic`
    - `Username`: `<ddns username>`
    - `Password`: `<ddns password>`
    - `API Endpoint`: `https://dns.eu.ovhapis.com/nic/update?system=dyndns&hostname=$HOSTNAME&myip=$IP`

### OVH configuration
  - For `<domain from OVH>` select `DynHost` tab
    - `Manage access` -> `Create a username`:
      - `The username suffix`: `<ddns username>`
      - `Sub-domain`: `<sub domain>` or `*(for all domains)`
      - `Password`: `<ddns password>`
    - `Add a DynHost record`:
      - `Sub-domain`: `<sub domain>` or `empty (for <domain from OVH>)`
      - `Current public IP`: `<public ip>`


## NextCloud client (optional)
### Android
- `Settings`:
  - `General` -> `Data storage location`: `<local storage>`
  - `Details` -> `Show app switcher`: `False`
  - `Sync` -> `Internal two way sync` -> `Enable two way sync`: `True`
- For each folder you want to have offline press `three dots` -> `Details` -> `Sync`: `True`


## Jellyfin (optional)
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


## qBittorrent (optional)
### Configuration
- Password setup:
  - Stop app
  - Open configuration in TrueNas Shell:
  ```
  nano /mnt/pool/qbittorrent/config_storage/qBittorrent/qBittorrent.conf
  ```
  - Add line to `Preferences section`:
  ```
  WebUI\Password_PBKDF2="@ByteArray(ARQ77eY1NUZaQsuDHbIMCA==:0WMRkYTUWVT9wVvdDtHAjU9b3b7uB8NR1Gur2hmQCvCDpm39Q+PsJRJPaCU51dEiz+dTzh8qbPsL8WkFljQYFQ==)"
  ```
  - Start app
  - `default username`: `admin`
  - `password`: `adminadmin`
- Auth setup:
  - `Tool` -> `Options` -> `WebUI` -> `Authentication`:
    - Chnage login and password
    - `Bypass authentication for clients in whitelisted IP subnets`: `<local ip address>/24`


## Nginx Proxy Manager (deprecated)
### DataSets structure
- `nginx_proxy_manager` (`Generic` + `Compression Level`: `OFF`)
  - `certs_storage` (`App`)
  - `data_storage` (`App`)

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


## References
Postgres DataSet: https://github.com/truenas/apps/issues/790  
NextCloud Office: https://youtu.be/sHU6XZ_b2hw?t=991  
Data protection: https://www.youtube.com/watch?v=dP0wagQVctc  
Configure config.php: https://github.com/nextcloud-snap/nextcloud-snap/wiki/Configure-config.php
