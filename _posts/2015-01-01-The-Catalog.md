---
title: The Catalog Of Awesome Things
updated: 2017-12-21 21:25
---

**Last Updated:** 2017-Dec-21 21:25

**Table of Contents**
* TOC
{:toc}

<hr>

# Things I regularly forget

## Blue-Green Deploys with Static Sites + Object Storage

* Manage it via DNS changes. (i.e. DNS Blue, DNS Green. Swap when deploy completes. Max 1 Deploy / 48hrs.)
* I was an idiot for registering this domain via Hover instead of Namecheap which has an API for that.
* I probably need to build my own DNS infrastructure for this when I have time since its got multiple uses.

Otherwise, you have to use a Gatekeeper that knows the Blue v. Green buckets and handles matters accordingly. Probably should use Netlify at that point but being me...I'll probably build some halfbaked version. For practice + Netlify doesn't give me a chance to test my own infra stuff.

## SSH

Force Password Authentication For SSH for testing if you secured things correctly. You bloody nub.
```bash
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no <url>
```

### Config

_Multiple Github Accounts_
```bash
#kgodwin
Host github.com-kgodwin
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_kgodwin

#account
Host github.com-account
    HostName github.com
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_account
```

## CSS

Zebra Stripe Tables
```css
tbody tr:nth-child(odd) {
   background-color: #FEE;
}
```

## RSS Feeds

<https://news.ycombinator.com/rss>

<https://www.reddit.com/r/webdev+programming/.rss>

<http://feeds.reuters.com/reuters/topNews?format=xml>

<https://www.reddit.com/r/worldnews/.rss>

<https://www.reddit.com/r/politics/.rss>

<https://www.reddit.com/r/netsec+security/.rss>

<https://static.nvd.nist.gov/feeds/xml/cve/misc/nvd-rss.xml>

## FSTAB + cifs

**credentials**
```
username=?
password=?
```

**fstab**
```
//$SERVER/$PATH /media/$USER/$PATH cifs credentials=/home/$USER/.bastioncifs,iocharset=utf8,sec=ntlm,uid=$USER,gid=$USER,user,noauto,x-systemd.automount 0 0
```


<hr>

# EDC

[Sneak 2017](https://sneak.berlin/20170107/current-daily-carry/)

[0x00](https://0x00sec.org/t/the-hackers-edc-bag/1651)

[MaddieDog](https://imgur.com/a/uhDyJ)

<hr>

# OSS Lists (Github)

## Things I tend to need from time to time

## Cli.Fyi

* <https://cli.fyi/> / mostly for the ip info
* <https://github.com/daveearley/cli.fyi>

## DNS

**LAN**
* <https://pi-hole.net/>

**Authoritative**
* <https://github.com/abh/geodns>
* <https://github.com/powerdns>

## Lists

[Full Stack Python](https://www.fullstackpython.com)

[PHP The Right Way](http://www.phptherightway.com/)

[PHP Static Analysis](https://github.com/exakat/php-static-analysis-tools)

[Awesome Selfhosted](https://github.com/Kickball/awesome-selfhosted)

[Awesome Development](https://github.com/sindresorhus/awesome)

<hr>

# Hosting (Personally used)

## Dedicated Vendors
_Recommended Vendors. Hobby=Cheap and OK._

| Rating | Vendor     |
|--------|------------|
| Hobby  | WholesaleInternet / Nocix / Datashack / et al (Least reliable but best otherwise) |
| Hobby  | Joe's Datacenter (More stable than Wholesale but not by much.) |
| Hobby  | Online.NET (Bad network outside EU) |
| Hobby  | Hetzner (Picky and easy to knock over) |
| Hobby  | OVH (Prod if you have good failover systems in place. Its just they go down and stay down for awhile.) |
| Hobby  | Worldstream.nl (Its been _years_.) |
| Prod   | ReliableServers |
| Prod   | HiVelocity |
| Prod   | Voxility |

## VPS Vendors
_Recommended Vendors are **Bold**_

| Price  | Power     | Rating | Vendor |
|--------|-----------|--------|--------|
| $3.5/mo| 1vCPU/1GB | Hobby  | **BuyVM** _Anycast_ 	|
| $4.5/mo| 1vCPU/2GB | Hobby  | **OVH** 				|
| €4/mo  | 1vCPU/1GB | Hobby  | Hetzner		            |
| $10/mo | 1vCPU/1GB | Hobby  | Amazon Lightsail 		|
| €3/mo  | 1vCPU/2GB | Hobby  | Scaleway                |
| $5/mo  | 1vCPU/1GB | Hobby  | **Linode** _LB_			|
| $5/mo  | 1vCPU/1GB | Prod   | **Vultr** 				|
| $10/mo | 1vCPU/1GB | Prod   | **Digital Ocean** _LB_ 	|

_Scaleway:_ Its just not as good as their dedicated Online.net brand (which is basically always out of stock now).

_Note About Linode:_ Do not use for financial transactions or other security-is-critical functions. Their security is subpar and they are not good about disclosure. But, for some random mobile app that doesn't have a massive honeypot of $$ if its exploited, its fine.


## Object Storage
_Recommended Vendors are **Bold**_

| Storage    | Transfer  | Rating     | Vendor |
|------------|-----------|------------|-----|
| $0.005/GB  | $0.02/GB  | Hobby | [B2 Backblaze](https://www.backblaze.com/b2/cloud-storage-pricing.html) |
| $0.0112/GB | $0.011/GB | Hobby | [**OVH Swift**](https://www.ovh.com/us/public-cloud/storage/object-storage/) | 
| $0.02/GB   | $0.01/GB  | Production | [**DO Spaces**](https://www.digitalocean.com/community/tutorials/an-introduction-to-digitalocean-spaces) |
| $0.023/GB  | $0.10/GB  | Production | [AWS S3](https://aws.amazon.com/s3/pricing/) |
| $0.026/GB  | $0.12/GB  | Production | [GCP](https://cloud.google.com/container-registry/pricing) |
|------------|-----------|------------|-----|

_DO Spaces:_ They don't support index.html yet, so not useful for say...a blog. Good object storage otherwise.

## Looking Glasses

<http://lon-gb-ping.vultr.com>

<http://wa-us-ping.vultr.com>

<http://lax-ca-us-ping.vultr.com>

<http://nj-us-ping.vultr.com>

<http://hnd-jp-ping.vultr.com>

<http://syd-au-ping.vultr.com>

<https://fra-de-ping.vultr.com>

<https://sgp-ping.vultr.com>

<https://lg.ovh.net/>

<https://lg.he.net/>

<http://www.cogentco.com/en/network/looking-glass>

<http://nj-test.buyvm.net/>

<http://speedtest.lv.buyvm.net/>

<http://lu-test.buyvm.net/>

<http://lax.meanservers.com/lg/>

<http://den.meanservers.com/lg/>

<https://www.ultratools.com/tools/lookingGlassTools>

<http://www.bgplookingglass.com>

<https://github.com/telephone/LookingGlass>

<hr>