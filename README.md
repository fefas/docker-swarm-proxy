# Docker Swarm Proxy

This repository aims to provide a reverse proxy for docker containers which are
webservices and are running on a swarm cluster. It also to provindes an
automated SSL.

> Need work

How to setup:

```
$ docker network create --driver overlay proxy

$ docker service create --name proxy-service-listener \
    --network proxy \
    --mount "type=bind,source=/var/run/docker.sock,target=/var/run/docker.sock" \
    -e DF_NOTIFY_CREATE_SERVICE_URL=http://proxy-haproxy:8080/v1/docker-flow-proxy/reconfigure \
    -e DF_NOTIFY_REMOVE_SERVICE_URL=http://proxy-haproxy:8080/v1/docker-flow-proxy/remove \
    --constraint 'node.role==manager' \
    fefas/swarm-proxy-service-listener

$ docker service create --name proxy-haproxy \
    -p 80:80 \
    -p 443:443 \
    --network proxy \
    -e LISTENER_ADDRESS=proxy-service-listener \
    -e SERVICE_NAME=proxy-haproxy \
    fefas/swarm-proxy-haproxy

$ docker service create --name proxy-ssl-provider \
    --label com.df.notify=true \
    --label com.df.distribute=true \
    --label com.df.servicePath=/.well-known/acme-challenge \
    --label com.df.port=80 \
    -e DOMAIN_1="('<your-domain>')" \
    -e CERTBOT_EMAIL="<your-email>" \
    -e PROXY_ADDRESS="proxy-haproxy" \
    -e CERTBOT_CRON_RENEW="('0 3 * * *')" \
    --network proxy \
    --mount type=bind,source=<some-path>/ssl-provider/letsencrypt,destination=/etc/letsencrypt \
    fefas/swarm-proxy-ssl-provider
```
