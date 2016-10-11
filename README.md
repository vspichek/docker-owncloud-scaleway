# docker-owncloud-scaleway
An example of setting up owncloud server on a Scaleway ARMv7 machine

This recipe uses `owncloud` image to provide owncloud functionality, `nginx` image to multiplex http connections and `docker-gen` image to watch docker for images that expose http <br/>
Performance (sqlite vs MySQL/Postgres, (mem)cache, extra apache server behind nginx) and security are beyond scope of this example

On the target machine you need
* Prepare docker-compose.override.yml where you override VIRTUAL_HOST and CERT_NAME variables and define volumes)
* put your certificate named **shared.crt** and **shared.key** into `nginx_certs` (with optionally multiple SANs thus **shared**). [WoSign](https://buy.wosign.com/free/) issues free certificates with up to 20 SANs
* put supplied **nginx.tmpl** into `templates` (taken from [nginx-proxy](https://github.com/jwilder/nginx-proxy) with [fix](https://github.com/jwilder/nginx-proxy/issues/304#issuecomment-189775983))
* build `owncloud` docker image:
  - git clone https://github.com/docker-library/owncloud.git
  - \# ( cd owncloud; git reset --hard 37fb493d8bdd4e0d65285b0474d91bce5a8260b5; ) # if you want repeatability
  - sed -i 's/FROM php:5.6-apache/FROM armhf/php:5.6-apache/' owncloud/9.1/apache/Dockerfile
  - sed -i 's/x86_64-linux-gnu/arm-linux-gnueabihf/'          owncloud/9.1/apache/Dockerfile
  - docker build -t armhf-owncloud:9.1-apache owncloud/9.1/apache
* **docker-compose up -d**

Depending on your configuration you may need to add following to `nginx_conf`
> server_names_hash_bucket_size  64;

This uses images available in Docker Hub<br/>
It's your responsibility to build and maintain set images that you can trust. Modify docker-compose.yml correspondingly 

[//]: <> (chown -R www-data:www-data /d/oc/data/vspichek/files/Video/)
[//]: <> (docker exec -u www-data owncloud php occ  files:scan --all)