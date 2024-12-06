# Home NAS
## TrueNAS
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

## Links:
Postgres dataset: https://github.com/truenas/apps/issues/790
