# autodiscover-email-settings

[![Docker Pulls](https://img.shields.io/docker/pulls/zikeji/autodiscover-email-settings.svg)](https://hub.docker.com/r/weboaks/autodiscover-email-settings/) [![Docker layers](https://images.microbadger.com/badges/image/zikeji/autodiscover-email-settings.svg)](https://microbadger.com/images/weboaks/autodiscover-email-settings)

This service is created to autodiscover your provider email settings.

It provides IMAP/SMTP Autodiscover capabilities on Microsoft Outlook/Apple Mail, Autoconfig capabilities for Thunderbird, and Configuration Profiles for iOS/Apple Mail.

### DNS settings

```
autoconfig              IN      A      {{$AUTODISCOVER_IP}}
autodiscover            IN      A      {{$AUTODISCOVER_IP}}
@                       IN      TXT     "mailconf=https://autoconfig.{{$DOMAIN}}/mail/config-v1.1.xml"
_imaps._tcp             IN      SRV    0 0 993 {{IMAP_DOMAIN}}.
_submission._tcp        IN      SRV    0 0 465 {{SMTP_DOMAIN}}.
_autodiscover._tcp      IN      SRV    0 0 443 autodiscover.{{$DOMAIN}}.
```

Replace above variables with data according to this table

| Variable        | Description                            |
| --------------- | -------------------------------------- |
| AUTODISCOVER_IP | IP of the autoconf server              |
| DOMAIN          | The domain name of the autoconf server |
| IMAP_DOMAIN     | The domain name of your IMAP server    |
| SMTP_DOMAIN     | The domain name of your SMTP server    |

---

### Usage

[traefik](https://github.com/containous/traefik) can proxy your containers on docker, on docker swarm, and on a wide range of orchestrators

Checks the Host header to populate the domain in the config, making it usable for deploying on multiple domains that all have a single IMAP/SMTP endpoint. Fallsback to the PROVIDER environment variable if no host header is found.

#### docker

```yaml
version: '2'

services:
  autodiscover:
    image: zikeji/autodiscover-email-settings:latest
    environment:
    - PROVIDER=domain.com
    - IMAP_HOST=imap.domain.com
    - IMAP_PORT=993
    - SMTP_HOST=smtp.domain.com
    - SMTP_PORT=465
    restart: unless-stopped
    labels:
      - "traefik.enable=true"
      - "traefik.http.services.autodiscover.loadbalancer.server.port=8000"
      - "traefik.http.routers.autodiscover.entrypoints=http"
      - "traefik.http.routers.autodiscover.rule=Host(`autodiscover.domain.com`) || Host(`autoconfig.domain.com`)"
      - "traefik.http.routers.autodiscover-secure.entrypoints=https"
      - "traefik.http.routers.autodiscover-secure.rule=Host(`autodiscover.domain.com`) || Host(`autoconfig.domain.com`)"
      - "traefik.http.routers.autodiscover-secure.tls=true"
      - "traefik.http.routers.autodiscover-secure.tls.certresolver=http"
```

#### docker swarm

```yaml
version: '3'

services:
  autodiscover-domain-com:
    image: zikeji/autodiscover-email-settings:latest
    environment:
    - PROVIDER=domain.com
    - IMAP_HOST=imap.domain.com
    - IMAP_PORT=993
    - SMTP_HOST=smtp.domain.com
    - SMTP_PORT=465
    deploy:
      replicas: 1
      labels:
        - "traefik.enable=true"
        - "traefik.http.services.autodiscover.loadbalancer.server.port=8000"
        - "traefik.http.routers.autodiscover.entrypoints=http"
        - "traefik.http.routers.autodiscover.rule=Host(`autodiscover.domain.com`) || Host(`autoconfig.domain.com`)"
        - "traefik.http.routers.autodiscover-secure.entrypoints=https"
        - "traefik.http.routers.autodiscover-secure.rule=Host(`autodiscover.domain.com`) || Host(`autoconfig.domain.com`)"
        - "traefik.http.routers.autodiscover-secure.tls=true"
        - "traefik.http.routers.autodiscover-secure.tls.certresolver=http"
```

### Credits

Spinoff of [autodiscover.xml](https://github.com/sylvaindumont/autodiscover.xml#credits) by [sylvaindumont](https://github.com/sylvaindumont/).

### Notes

The above autoconfiguration methods assume the following:

* Username: `{{email}}` (Entire email address)
* Encryption: SSL/TLS

### License

This project is distributed under the [MIT License](LICENSE)
