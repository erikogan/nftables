# Nftables

1. [Overview](#overview)
2. [Role Variables](#role-variables)
     * [OS Specific Variables](#os-specific-variables)
     * [Rules Dictionaries](#rules-dictionaries)
3. [Example Playbook](#example-playbook)
4. [Configuration](#configuration)
5. [Development](#development)
5. [License](#license)
6. [Author Information](#author-information)

## Overview

A role to manage Nftables rules and packages.

Highly inspired by [Mike Gleason firewall role][mikegleasonjr firewall github] (3 levels of rules definition and template), thanks !

## Role Variables

* **nft_pkg_manage** : If `nftables` package(s) should be managed with this role [default : `true`].
* **nft_pkg_state** : State of new `nftables` package(s) [default : `installed`].
* **nft_main_conf_path** : Main configuration file loaded by systemd unit [default : `/etc/nftables.conf`].
* **nft_main_conf_content** : Template used to generate the previous main configuration file [default : `etc/nftables.conf.j2`].
* **nft_input_conf_path** : Input configuration file include in main configuration file [default : `/etc/nftables.d/inet-filter.nft`].
* **nft_input_conf_content** : Template used to generate the previous input configuration file [default : `etc/nftables.d/inet-filter.nft.j2`].
* **nft_global_default_rules** : Set default rules for `global` chain. Other chains will jump to `global` before apply their specific rules.
* **nft_global_group_rules** : You can add `global` rules or override those defined by **nft_global_default_rules** for a group.
* **nft_global_host_rules:** : Hosts can also add or override `global` rules.
* **nft_input_default_rules** : Set default rules for `input` chain.
* **nft_input_group_rules** : You can add `input` rules or override those defined by **nft_input_default_rules** for a group.
* **nft_input_host_rules:** : Hosts can also add or override `input` rules.
* **nft_service_manage** : If `nftables` service should be managed with this role [default : `true`].
* **nft_service_name** : `nftables` service name [default : `nftables`].

### OS Specific Variables

Please see default value by Operating System file in [vars][vars directory] directory.

* **nft_pkg_list** : The list of package(s) to provide `nftables`.

### Rules Dictionaries

Each type of rules dictionaries will be merged and rules will be applied in the alphabetical order of the keys (the reason to use 000 to 999 as prefix). So :
  * **nft_*_default_rules** : Define default rules for all nodes. You can define it in `group_vars/all`.
  * **nft_*_group_rules** : Can add rules and override those defined by **nft_*_default_rules**. You can define it in `group_vars/webservers`.
  * **nft_*_host_rules** : Can add rules and override those define by **nft_*_default_rules** and **nft_*_group_rules**. You can define it in `host_vars/www.local.domain`.

`defaults/main.yml`:

``` yml
# rules
nft_global_default_rules:
  000 state management:
    - ct state established,related accept
    - ct state invalid drop
nft_global_group_rules: {}
nft_global_host_rules: {}

nft_input_default_rules:
  000 policy:
    - type filter hook input priority 0; policy drop;
  001 global:
    - jump global
nft_input_group_rules: {}
nft_input_host_rules: {}
```

Those default will generate the following configuration :
```
#!/usr/sbin/nft -f
# Ansible managed


# clean
flush ruleset

table inet firewall {
	chain global {
		# 000 state management
		ct state established,related accept
		ct state invalid drop
	}
	chain input {
		type filter hook input priority 0; policy drop;
		jump global
	}
	chain output {
		type filter hook output priority 0;
		jump global
	}
}
```

And you get the same result by displaying the ruleset on the host : `$ nft list ruleset` :

```
table inet firewall {
	chain global {
		ct state established,related accept
		ct state invalid drop
	}

	chain input {
		type filter hook input priority 0; policy drop;
		jump global
	}

	chain output {
		type filter hook output priority 0; policy accept;
		jump global
	}
}
```

## Example Playbook

* Manage Nftables with defaults vars :

``` yml
- hosts: serverXYZ
  roles:
    - role: ipr-cnrs.nftables
```

* Use default rules with allow ICMP and count dropped input packets :

`group_vars/all` :

``` yaml
nft_global_group_rules:
  002 icmp:
    - ip protocol icmp accept
```

`group_vars/first_group` :

``` yaml
nft_input_group_rules:
  999 count policy packet:
    - counter
```

## Configuration

This role will :
* Install `nftables` on the system.
* Generate a default configuration file loaded by systemd unit.
* Generate input rules file include called by the main configuration file.
* Restart `nftables` service.

## Development

This source code comes from our [Gogs instance][nftables source] and the [Github repo][nftables github] exist just to be able to send the role to Ansible Galaxy…

But feel free to send issue/PR here :)

Thanks to this [hook][gogs to github hook], Github automatically got updates from our [Gogs instance][nftables source] :)

## License

[WTFPL][wtfpl website]

## Author Information

Jérémy Gardais
* Source : [on IPR's Gogs][nftables source]
* [IPR][ipr website] (Institut de Physique de Rennes)

[gogs to github hook]: https://stackoverflow.com/a/21998477
[nftables source]: https://git.ipr.univ-rennes1.fr/cellinfo/ansible.nftables
[nftables github]: https://github.com/ipr-cnrs/nftables
[wtfpl website]: http://www.wtfpl.net/about/
[ipr website]: https://ipr.univ-rennes1.fr/
[mikegleasonjr firewall github]: https://github.com/mikegleasonjr/ansible-role-firewall
