# Customizations

The following sections describe optional customizations you can make to further modify your Islandora site.

## Modifying settings.php

TODO

## Modifying robots.txt

TODO

## Nginx customizations

### Blocking IP Addresses

TODO

### Blocking by User Agent

TODO

## Traefik customizations

### Using Your Certs Instead of LetsEncrypt

TODO

### TLS Settings

Traefik allows you to modify security settings by doing things like setting a minimum TLS version or specifying cipher suites. Isle Site Template ships with two versions of the TLS settings file, one [for development](https://github.com/Islandora-Devops/isle-site-template/blob/main/dev-tls.yml), and one [for production](https://github.com/Islandora-Devops/isle-site-template/blob/main/prod-tls.yml).

These files are mounted as volumes in the Traefik containers, so you can modify them as desired.

More information is available in the [Traefik documentation](https://doc.traefik.io/traefik/https/tls/#tls-options)