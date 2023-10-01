# Template for fail2ban configuration

## Components location

**Local** components are suffixed with `local` to not be overrided by [defaults fail2ban components](https://manpages.debian.org/stable/fail2ban/jail.conf.5.en.html#CONFIGURATION_FILES_FORMAT).

- Rule/jail directives are stored in the `/etc/fail2ban/jail.d` folder.

- Filters are stored in the `/etc/fail2ban/filter.d` folder.

- Ban-actions are stored in the `/etc/fail2ban/action.d` folder.

## Activating service

Don't forget to activate the service to avoid not restarting after a reboot

```bash
sudo systemctl enable fail2ban
```

## General rule attributes

<https://github.com/fail2ban/fail2ban/blob/a9b30eb86ea4367e7464c90f517b1e1da9c88020/config/jail.conf>

enabled = true

ignoreip = 192.168.1.0/24

It will enable the rule to be effective and ignore all fails made from a local ip address of the standard private network.

> **Warning**
> Your private network is supposed to be secure !

`bantime.increment` allows to use database for searching of previously banned ip's to increase a default ban time.

`bantime.rndtime` is the max number of seconds using for mixing with random time to prevent "clever"
botnets calculate exact time IP can be unbanned again.

`bantime.factor` is a coefficient to calculate exponent growing of the formula or common multiplier,
 default value of factor is 1 and with default value of formula, the ban time
 grows by 1, 2, 4, 8, 16 ... so it is multiplied by 2 by default
 
 bantime.formula = ban.Time * (1<<(ban.Count if ban.Count<20 else 20)) * banFactor

> **Note**
> No need to use `recidive` jail with `bantime.increment`.

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

It use a `failregex` to match errors entry in the nginx-proxy-manager log files.

### Nginx proxy action

Because docker system doesn't work on the `default` iptable chain and has is **own** chain, the ufw banaction is [ineffective](https://docs.docker.com/network/iptables/#add-iptables-policies-before-dockers-rules).
To outpass this, I've [searched](https://blog.lrvt.de/fail2ban-with-nginx-proxy-manager/) fo a `docker-action` who use the `DOCKER-USER` chain of iptable to ban properly a malicious ip.

## Acknowledgments

Thanks to [LRVT](https://github.com/l4rm4nd) for his [repository](https://github.com/l4rm4nd/F2BFilters) and his [blog](https://blog.lrvt.de/) with plenty of good advices and knowledge.
