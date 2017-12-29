---
title: MarionetteDNS.com aka DIY GeoDNS
updated: 2017-12-28 19:00
---

This is a test project to give me a chance to play around with Anycast, GeoDNS, and DNS for situations that aren't 24/7/365 mission critical. In other words, for hobby sites for various people. Just because it uses MarionetteDNS does not mean I own it.

If you are serious about using a CDN, you probably should _just_ use a CDN. The main reason I'm not is I've got some backend complexity (i.e. datacenter level failover) that the cheap/easy CDNs don't really support correctly (if at all). For instance, I'm using some _very_ cheap OVH storage that I'm replicating between NA & EU zones (partially due to reliability, partially due to testing object store replication software). 

That might seem _odd_ given I could just use S3 but this is cheaper (even before you factor in AWS $.09/GB bandwidth pricing).

The flow is basically MarionetteDNS -> PoP -> Origin A or B.


Please note this is an update to an old site / project so stack.guide doesn't actually _go_ anywhere.

## Note

4.0.X is bugged with regions and the Maxmind GeoIP datasets. https://github.com/PowerDNS/pdns/issues/5255

Given that I'm basically just running 4 PoPs this isn't a serious issue since North America is really the only place it "matters" and I'm just splitting the difference. (i.e. NY replies with US East and LV replies with US West)

The other bit is, honestly, below the Continent level...GeoDNS can get pretty iffy unless you pay for a quality data source. When the wrong countries, regions, and/or cities get returned it impacts your routing.

## The Basics

1) Setup the glue records on your DNS registrar (which really varies by registrar too much to give specifics).

2) Install PowerDNS: https://doc.powerdns.com/authoritative/installation.html

3) Setup the GeoIP Backend: https://doc.powerdns.com/authoritative/backends/geoip.html

4) Confirm you've set it up correctly: http://www.dnsstuff.com/tools , https://dnschecker.org/ , and https://check-host.net/

## Operating Cost

~$31/month

To be honest, its probably cheaper to just run everything through 1 Anycast'd trio of BuyVM VMs. However, given this is more of a lab / experimental setup that I've allowed a few people to use...it makes sense to stick with this configuration.

## The Nodes (BuyVM)

#### ns1.marionettedns.com / 198.251.90.221
* lu1.marionettedns.com
* ny1.marionettedns.com
* lv1.marionettedns.com

#### ns2.marionettedns.com / 198.251.90.222
* lu2.marionettedns.com
* ny2.marionettedns.com
* lv2.marionettedns.com

## Example Nginx Config

_Please note, you have to do special things to make the letsencrypt bits work as intended. I used to use a regular SSL cert but I'm experimenting with multiple hosts and letsencrypt at the moment, hence the variation. stack.guide is basically a test domain I'm not using for anything serious at the moment._

```nginx
proxy_cache_path  /opt/cache/stack.guide  levels=1:2  keys_zone=STACK_GUIDE:10m inactive=24h  max_size=10g;

# Reverse if one is closer than the other.
upstream backend {
    server storage.bhs3.cloud.ovh.net:443 max_fails=2 fail_timeout=10s;
    server storage.gra3.cloud.ovh.net:443 backup;
}

server {
    listen 80;
    listen [::]:80;

    server_name stack.guide;

    root /var/www/html;
    access_log /var/log/nginx/stack.guide.access.log;

    # allow lets encrypt locally
    location ~ /.well-known {
        try_files $uri $uri/ =404;
        allow all;
    }

    location / {
        return 301 https://$server_name$request_uri;
    }

}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    server_name stack.guide;

    root /var/www/html;
    access_log /var/log/nginx/https.stack.guide.access.log;

    ssl_certificate /etc/letsencrypt/live/stack.guide/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/stack.guide/privkey.pem;
    ssl_session_cache shared:SSL:45m;
    ssl_session_timeout 5m;

    # allow lets encrypt locally
    location ~ /.well-known {
        try_files $uri $uri/ =404;
        allow all;
    }

    location / {
        # target
        proxy_pass https://backend/v1/AUTH_0c44062f5afd49d49a99231231e78ead/testing/;
        proxy_ssl_session_reuse on;

        # given its just images, no real reason to log this
        access_log        off;
        log_not_found     off;

        # intercept errors
        proxy_intercept_errors on;

        # prevent non-get requests
        proxy_method GET;

        # proxy dns and have it timeout to keep things from being cached too long
        resolver 8.8.8.8 8.8.4.4;
        resolver_timeout 10s;

        # proxy headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        # Proxy Con info
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_buffering off;
        client_max_body_size 0;
        proxy_read_timeout 15s; # If it takes longer tha 15s, something is seriously wrong.
        proxy_redirect off;

        # If its down, continue using cache for 1 days
        proxy_cache             STACK_GUIDE;
        proxy_cache_valid       200  1d;
        proxy_cache_valid       404 302 1m;
        proxy_cache_use_stale   error timeout invalid_header updating http_500 http_502 http_503 http_504;

        # Don't cache the long tail, require 2+ views
        proxy_cache_min_uses 2;

    }
}
            
```

## Example PowerDNS Config

### LV

```yaml
####################################################### 
#
# POP Design: ($10/month, 4x $2.50 VPS)
# .99 = US East     = Vultr New Jersey
# .42 = US West     = Vultr Silicon Valley
# .3 = Europe       = Vultr Paris
# .4 = Australia    = Vultr Sydney
#
#######################################################
domains:
- domain: stack.guide
  ttl: 60
  records:
    na.geo.stack.guide:
        - soa: ns1.marionettedns.com root@marionettedns.com 2017122601 7200 3600 1209600 3600
        - a: 127.0.0.42
        - txt: "NA East -> Vultr New Jersey"
    eu.geo.stack.guide:
        - soa: ns1.marionettedns.com root@marionettedns.com 2017122601 7200 3600 1209600 3600
        - a: 127.0.0.3
        - txt: "Europe -> Vultr Paris"
    oc.geo.stack.guide:
        - soa: ns1.marionettedns.com root@marionettedns.com 2017122601 7200 3600 1209600 3600
        - a: 127.0.0.4
        - txt: "Oceania -> Vultr Australia"
    as.geo.stack.guide:
        - soa: ns1.marionettedns.com root@marionettedns.com 2017122601 7200 3600 1209600 3600
        - a: 127.0.0.42
        - txt: "Asia -> Vultr Silicon Valley"
    af.geo.stack.guide:
        - soa: ns1.marionettedns.com root@marionettedns.com 2017122601 7200 3600 1209600 3600
        - a: 127.0.0.3
        - txt: "Africa -> Vultr Paris"
    sa.geo.stack.guide:
        - soa: ns1.marionettedns.com root@marionettedns.com 2017122601 7200 3600 1209600 3600
        - a: 127.0.0.99
        - txt: "South America -> Vultr New Jersey"
    unknown.geo.stack.guide:
        - soa: ns1.marionettedns.com root@marionettedns.com 2017122601 7200 3600 1209600 3600
        - a: 127.0.0.3
        - txt: "Antarctica -> Vultr Paris"
    "*.stack.guide":
        - a: 127.0.0.53
        - txt: "No idea."
  services:
    stack.guide: ['%cn.geo.stack.guide', 'unknown.geo.stack.guide']
```

### NY & EU

```yaml
####################################################### 
#
# POP Design: ($10/month, 4x $2.50 VPS)
# .99 = US East     = Vultr New Jersey
# .42 = US West     = Vultr Silicon Valley
# .3 = Europe       = Vultr Paris
# .4 = Australia    = Vultr Sydney
#
#######################################################
domains:
- domain: stack.guide
  ttl: 60
  records:
    na.geo.stack.guide:
        - soa: ns1.marionettedns.com root@marionettedns.com 2017122601 7200 3600 1209600 3600
        - a: 127.0.0.99
        - txt: "NA East -> Vultr New Jersey"
    eu.geo.stack.guide:
        - soa: ns1.marionettedns.com root@marionettedns.com 2017122601 7200 3600 1209600 3600
        - a: 127.0.0.3
        - txt: "Europe -> Vultr Paris"
    oc.geo.stack.guide:
        - soa: ns1.marionettedns.com root@marionettedns.com 2017122601 7200 3600 1209600 3600
        - a: 127.0.0.4
        - txt: "Oceania -> Vultr Australia"
    as.geo.stack.guide:
        - soa: ns1.marionettedns.com root@marionettedns.com 2017122601 7200 3600 1209600 3600
        - a: 127.0.0.42
        - txt: "Asia -> Vultr Silicon Valley"
    af.geo.stack.guide:
        - soa: ns1.marionettedns.com root@marionettedns.com 2017122601 7200 3600 1209600 3600
        - a: 127.0.0.3
        - txt: "Africa -> Vultr Paris"
    sa.geo.stack.guide:
        - soa: ns1.marionettedns.com root@marionettedns.com 2017122601 7200 3600 1209600 3600
        - a: 127.0.0.99
        - txt: "South America -> Vultr New Jersey"
    unknown.geo.stack.guide:
        - soa: ns1.marionettedns.com root@marionettedns.com 2017122601 7200 3600 1209600 3600
        - a: 127.0.0.3
        - txt: "Antarctica -> Vultr Paris"
    "*.stack.guide":
        - a: 127.0.0.53
        - txt: "No idea."
  services:
    stack.guide: ['%cn.geo.stack.guide', 'unknown.geo.stack.guide']
```