# docker-nextcloud

### TL;DR

```
git clone https://github.com/victorskl/docker-nextcloud.git nextcloud
cd nextcloud
mkdir -p data/nextcloud/data
chown 33:root data/nextcloud/data

cp env.sample .env
cp docker-compose.override.yaml.sample docker-compose.override.yaml

docker-compose up -d
docker-compose ps
docker-compose down
```

### Reverse Proxy

#### HAProxy
```
haproxy -v
HA-Proxy version 1.5.18 2016/05/10

vi /etc/haproxy/haproxy.cfg

    frontend http
        bind *:80
        default_backend redirect-https
    
    backend redirect-https
        reqadd X-Forwaded-Proto:\ http
        redirect scheme https code 301 if !{ ssl_fc }
    
    frontend https
        bind *:443 ssl crt /etc/haproxy/certs/example.com.pem
        http-request set-header X-Forwarded-Host %[req.hdr(Host)]
        reqadd X-Forwaded-Proto:\ https
        http-response set-header Strict-Transport-Security max-age=15552000;\ includeSubDomains;\ preload;
        default_backend nextcloud
    
    backend nextcloud
       server nc1 127.0.0.1:8080
```

#### Config

```
docker network inspect nextcloud_default | grep Subnet
    "Subnet": "172.25.0.0/16",

vi data/nextcloud/app/config/config.php

    'trusted_domains' => array (0 => 'localhost', 1 => 'cloud.example.com'),
    'trusted_proxies' => array('172.25.0.0/16'),
    'forwarded_for_headers' => array('HTTP_X_FORWARDED', 'HTTP_FORWARDED_FOR'),
```

## REF

- https://hub.docker.com/_/nextcloud
- https://hub.docker.com/_/mariadb
- https://hub.docker.com/_/redis
- https://hub.docker.com/_/adminer
