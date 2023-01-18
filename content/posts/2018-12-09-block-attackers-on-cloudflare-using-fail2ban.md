---
title: 'Block attackers on CloudFlare using Fail2Ban'
date: '2018-12-09T00:11:08-05:00'
author: Adrien Poupa
url: /block-attackers-on-cloudflare-using-fail2ban/
---

Recently, I had to counter a layer 7 HTTP Flood DDoS attack on my server, that is using CloudFlare. I started by setting up Fail2Ban using the Nginx logs, and Fail2Ban would ban attackers but they would still be able to hit my server. I finally understood that since they were passing through CloudFlare, I had to block them at a higher level, CloudFlare itself. Fortunately, CloudFlare offers a firewall and an API to block offenders.

Thus, the solution I found is to analyze Nginx’s logs with Fail2Ban, and trigger a ban once a certain threshold is met. We will ban the user from the server but also from accessing CloudFlare using their REST API.

First, for the logs to be exploitable, make sure that Nginx’s configuration contains the following code snippet, that transforms CloudFlare’s IPs into the real visitor’s IP:

```
    # CloudFlare
    set_real_ip_from 103.21.244.0/22;
    set_real_ip_from 103.22.200.0/22;
    set_real_ip_from 103.31.4.0/22;
    set_real_ip_from 104.16.0.0/12;
    set_real_ip_from 108.162.192.0/18;
    set_real_ip_from 131.0.72.0/22;
    set_real_ip_from 141.101.64.0/18;
    set_real_ip_from 162.158.0.0/15;
    set_real_ip_from 172.64.0.0/13;
    set_real_ip_from 173.245.48.0/20;
    set_real_ip_from 188.114.96.0/20;
    set_real_ip_from 190.93.240.0/20;
    set_real_ip_from 197.234.240.0/22;
    set_real_ip_from 198.41.128.0/17;
    set_real_ip_from 2400:cb00::/32;
    set_real_ip_from 2405:b500::/32;
    set_real_ip_from 2606:4700::/32;
    set_real_ip_from 2803:f800::/32;
    set_real_ip_from 2c0f:f248::/32;
    set_real_ip_from 2a06:98c0::/29;
    real_ip_header    CF-Connecting-IP;
```

The list of IPs often change; the latest one can be found at <https://www.cloudflare.com/ips/>

Then, let’s create zones that we can reuse in our virtual hosts. Zones are buckets that allow to limit one’s requests to a certain threshold. The following code snippet limits each user to 10 requests per second and creates a second zone that limits POST requests to 2 per second. Feel free to tweak those values.

```
    limit_conn_zone $binary_remote_addr zone=limit_conn_zone_var:10m;
    limit_req_zone $binary_remote_addr zone=limit_req_zone_var:10m rate=10r/s;

    # Maps ip address to $limit variable if request is of type POST
     map $request_method $limit {
       default         "";
       POST            $binary_remote_addr;
    }

    # Creates 10mb zone in memory for storing binary ips, limit POST to 2r/s
    limit_conn_zone $limit zone=post_limit_conn_zone:10m;
    limit_req_zone $limit zone=post_limit:10m rate=2r/s;
```

Then, in the virtual host file, add the following, that limits the number of connections to 10 with a burst of 20 for the number of requests (first loading of the page with all the static content).

```
        limit_conn limit_conn_zone_var 10;
        limit_req zone=limit_req_zone_var burst=20 nodelay;

        limit_conn post_limit_conn_zone 10;
        limit_req zone=post_limit nodelay;
```

Next, let’s install Fail2Ban. I would recommend installing it from the source because the default version available in Debian in ancient and does not support IPv6 addresses well (0.9.x on Debian Stretch when the first version that supports IPv6 addresses is 0.10). The instructions for version 0.11 can be found on [GitHub](https://github.com/fail2ban/fail2ban).

Even though Fail2Ban supports CloudFlare firewall out of the box, I had issues running it until I found [this repository.](https://github.com/wpkc/fail2ban-action-cloudflare-restv4) Inspired by [their article](https://www.kazimer.com/fail2ban-action-for-cloudflare-rest-api-v4/), copy [the following file](https://raw.githubusercontent.com/wpkc/fail2ban-action-cloudflare-restv4/master/action.d/cloudflare-restv4.conf) to your /etc/fail2ban/action.d folder. For reference, the file is the following:

```
# Fail2Ban action configuration file for CloudFlare REST API V4
#
# Author: Kazimer Corp
# Auther URL: https://www.kazimer.com
#
# This action depends on curl, python, and xargs.
#
# To get your CloudFlare Global API Key: https://dash.cloudflare.com/profile
#
# CloudFlare API firewall rules documentation: https://api.cloudflare.com/#user-level-firewall-access-rule-properties
#
# How to use:
# 	Add your CloudFlare email, API key, and the action call to the  [DEFAULT] section of your jail.local file:
#
# 		cfemail = user@example.com
# 		cfapikey = c2547eb745079dac9320b638f5e225cf483cc5cfdda41
#		action_cf_v4 = cloudflare-restv4[cfuser="%(cfemail)s", cfkey="%(cfapikey)s"]
#
#	Set the default action in the [DEFAULT] section, or override the default action in a jail:
#		action = %(action_cf_v4)s

[Definition]

# Option:  actionstart
# Notes.:  command executed once at the start of Fail2Ban.
# Values:  CMD
#
actionstart =

# Option:  actionstop
# Notes.:  command executed once at the end of Fail2Ban
# Values:  CMD
#
actionstop =

# Option:  actioncheck
# Notes.:  command executed once before each actionban command
# Values:  CMD
#
actioncheck =

# Option:  actionban
# Notes.:  command executed when banning an IP. Take care that the
#          command is executed with Fail2Ban user rights.
# Tags:    <ip>  IP address
#          <failures>  number of failures
#          <time>  unix timestamp of the ban time
# Values:  CMD
#
actionban = curl -s -X POST https://api.cloudflare.com/client/v4/user/firewall/access_rules/rules \
	-H "X-Auth-Email: <cfuser>" -H "X-Auth-Key: <cfkey>" -H "Content-Type: application/json" \
	--data '{"mode":"block","configuration":{"target":"ip","value":"<ip>"},"notes":"Banned by Fail2Ban"}'

# Option:  actionunban
# Notes.:  command executed when unbanning an IP. Take care that the
#          command is executed with Fail2Ban user rights.
# Tags:    <ip>  IP address
#          <failures>  number of failures
#          <time>  unix timestamp of the ban time
# Values:  CMD
#
actionunban = curl -s -X GET -H "X-Auth-Email: <cfuser>" -H "X-Auth-Key: <cfkey>" -H "Content-Type: application/json" \
	"https://api.cloudflare.com/client/v4/user/firewall/access_rules/rules?page=1&per_page=5&mode=block&configuration.target=ip&configuration.value=<ip>&notes=Banned by Fail2Ban&match=all&order=configuration.value&direction=desc" | \
	python -c "import sys, json; print json.load(sys.stdin)['result'][0]['id'];" | \
	xargs -I@@ curl -s -X DELETE -H "X-Auth-Email: <cfuser>" -H "X-Auth-Key: <cfkey>" -H "Content-Type: application/json" https://api.cloudflare.com/client/v4/user/firewall/access_rules/rules/@@

[Init]

# Declare your CF account e-mail in the [DEFAULT] section of your jail.local file.
# Example:
#	cfemail  = user@example.com

# Declare your CloudFlare Global API Key in the [DEFAULT] section of your jail.local file.
# Example:
#	cfapikey = c2547eb745079dac9320b638f5e225cf483cc5cfdda41
```

Then, get your CloudFlare API [by following the instructions](https://support.cloudflare.com/hc/en-us/articles/200167836-Where-do-I-find-my-Cloudflare-API-key-), and modify Fail2Ban’s jail.local configuration file as follows:

```
cfemail = youremail
cfapikey = yourapikey

# "ignoreip" can be an IP address, a CIDR mask or a DNS host. Fail2ban will not
# ban a host which matches an address in this list. Several addresses can be
# defined using space (and/or comma) separator.
ignoreip = 127.0.0.1/8 103.21.244.0/22 103.22.200.0/22 103.31.4.0/22 104.16.0.0/12 108.162.192.0/18 131.0.72.0/22 141.101.64.0/18 162.158.0.0/15 172.64.0.0/13 173.245.48.0/20 188.114.96.0/20 190.93.240.0/20 197.234.240.0/22 198.41.128.0/17
```

Add the following 2 jails for Nginx:

```
[nginx-req-limit]
enabled = true
filter = nginx-req-limit
action = iptables-multiport[name=ReqLimit-tcp, port="http,https", protocol=tcp]
         iptables-multiport[name=ReqLimit-udp, port="http,https", protocol=udp]
         cloudflare-restv4[cfuser="%(cfemail)s", cfkey="%(cfapikey)s"]
logpath = /var/log/nginx/error.log
          /var/log/nginx/access.log
findtime = 3600
bantime = 3600
maxretry = 20

[nginx-conn-limit]
enabled = true
filter = nginx-conn-limit
action = iptables-multiport[name=ConnLimit-tcp, port="http,https", protocol=tcp]
         iptables-multiport[name=ConnLimit-udp, port="http,https", protocol=udp]
         cloudflare-restv4[cfuser="%(cfemail)s", cfkey="%(cfapikey)s"]
logpath = /var/log/nginx/error.log
          /var/log/nginx/access.log
findtime = 3600
bantime = 3600
maxretry = 20
```

This will trigger the CloudFlare action when Fail2Ban detects an Nginx DDoS. The current values allow the attacker to be flagged by Nginx 20 times max every hour before being banned. You can add more log files to analyze. Also, the following line should be added to the \[recidive\] jail in the action section:

```
cloudflare-restv4[cfuser="%(cfemail)s", cfkey="%(cfapikey)s"]
```

And you’re all set! Reload Nginx and Fail2Ban, you should see bans appearing in your CloudFlare’s admin panel 🙂