# Template for fail2ban configuration

## Components location

**Local** components are suffixed with `local` to not be overrided by [defaults fail2ban components](https://manpages.debian.org/stable/fail2ban/jail.conf.5.en.html#CONFIGURATION_FILES_FORMAT).

- Rule/jail directives are stored in the `/etc/fail2ban/jail.d` folder.

- Filters are stored in the `/etc/fail2ban/filter.d` folder.

- Ban-actions are stored in the `/etc/fail2ban/action.d` folder.

## General rule attributes

enabled = true

ignoreip = 192.168.1.0/24

It will enable the rule to be effective and ignore all fails made from a local ip address of the standard private network.

> **Warning**
> Your private network is supposed to be secure !

## Ssh rule

For the ssh rule:

If in the `findtime` time windows , it find `maxretry + 1` log entries who triggers the **default** ssh filter (fail attempts), it will execute the `banaction` for the given ip address who will be active until the end of `bantime`.

## Recidive rule

For the recidive rule:

If in the `findtime` time windows , it find `maxretry + 1` log entries who triggers the **default** recidive filter (an ip who make fail attempts even after been banned by **another filter**), it will execute the `banaction` for the given ip address who will be active until the end of `bantime`.

## Nginx proxy rule

For the nginx proxy rule:

If in the `findtime` time windows , it find `maxretry + 1` log entries in the **custom** `logpath` files (see sections bellow) who triggers the **custom** recidive `filter` (see sections bellow), it will execute the **custom** `banaction` for the given ip address (see sections bellow) who will be active until the end of `bantime`.

### Nginx proxy logpath

Because I use [nginx proxy manager](https://nginxproxymanager.com/), I have several services running on docker reconducted by this reverse proxy and all connection events related to them are logged by it.

So the `logpath` variable point to these log files for each service.

### Nginx proxy filter

It use a `failregex` to match an 404 or 303 error entry in the nginx-proxy-manager log files.

### Nginx proxy action

Because docker system doesn't work on the `default` iptable chain and has is **own** chain, the ufw banaction is [ineffective](https://docs.docker.com/network/iptables/#add-iptables-policies-before-dockers-rules).
To outpass this, I've created a `docker-action` who use the `DOCKER-USER` chain of iptable to ban properly a malicious ip.
