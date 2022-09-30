## 1. Make db dump of current local magento db

make mysql database dump

## 2. Install warden

```
brew install warden
```

## 3. Put .env in project root

move the .env file into magento project root

## 4. Sign test domain certificate (TRAEFIK_DOMAIN in .env)

```
warden sign-certificate app.abbott.test
```

## 5. Pull latest warden SVC images and start them

```
warden svc pull
warden svc up --force-recreate
```

### Apple M1 Chip DNSMasq Fixes

#### DNSMasq Port Conflict

if you are unable to get to any warden test domains, you might have port conflict for 53

Reference: https://github.com/davidalger/warden/issues/262#issuecomment-748291589

5A. Edit the docker-compose for warden:

In `/usr/local/opt/warden/docker/docker-compose.yml`:

Change the port mapping for dnsmasq:

```
      - "127.0.0.1:5053:53/udp"
```

5B. Edit the test resolver:

In `vi /etc/resolver/test`:

Add the port:

```
nameserver 127.0.0.1
port 5053
```

5C. Restart dnsmasq:

```
warden svc restart dnsmasq
```

## 6. Pull latest warden images

```
warden env pull
warden env up --force-recreate`
```

### 6A. Apple M1 Image Slowness Fix

Some of the images used by warden OOTB are very slow on ARM64 architecture

Create `.warden/warden-env.darwin.yml` in project root and place following contents:

```xml
version: "3.5"
services:
  php-debug:
    image: ghcr.io/drpayyne/warden-php-m2-xdebug3:${PHP_VERSION}-deb
  php-fpm:
    image: ghcr.io/drpayyne/warden-php-m2:${PHP_VERSION}-deb
```


## 7. Import DB dump

```
cat magento_db.sql | warden db import
```


## 8. Warden things
When a project is setup using warden there are steps followed for setup. Here is documentation for reference:

**Warden - Magento 2**: https://docs.warden.dev/environments/magento2.html



**Elastic Search**
```
bin/magento config:set --lock-env catalog/search/engine elasticsearch7
bin/magento config:set --lock-env catalog/search/elasticsearch7_server_hostname elasticsearch
bin/magento config:set --lock-env catalog/search/elasticsearch7_server_port 9200
bin/magento config:set --lock-env catalog/search/elasticsearch7_index_prefix magento2
bin/magento config:set --lock-env catalog/search/elasticsearch7_enable_auth 0
bin/magento config:set --lock-env catalog/search/elasticsearch7_server_timeout 15
```

**Base Url**
```
bin/magento config:set --lock-env web/unsecure/base_url \
    "https://${TRAEFIK_SUBDOMAIN}.${TRAEFIK_DOMAIN}/"

bin/magento config:set --lock-env web/secure/base_url \
    "https://${TRAEFIK_SUBDOMAIN}.${TRAEFIK_DOMAIN}/"
```

**Offload header (if using varnish)**
```
bin/magento config:set --lock-env web/secure/offloader_header X-Forwarded-Proto
```

**Use Secure urls in frontend/admin + use rewrites**
```
bin/magento config:set --lock-env web/secure/use_in_frontend 1
bin/magento config:set --lock-env web/secure/use_in_adminhtml 1
bin/magento config:set --lock-env web/seo/use_rewrites 1
```

**Enable EAV indexer**
```
bin/magento config:set --lock-env catalog/search/enable_eav_indexer 1
```

**Disable static signing of assets**
```
bin/magento config:set --lock-env dev/static/sign 0
```

**Developer mode**
```
bin/magento deploy:mode:set -s developer
```

**Disable some caches**
```
bin/magento cache:disable block_html full_page
```

**Reindex**
```
bin/magento indexer:reindex
```

**Flush cache**
```
bin/magento cache:flush
```

## 9. View site in browser

url: https://app.abbott.test