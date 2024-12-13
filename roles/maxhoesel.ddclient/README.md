maxhoesel.ddclient
=========

A very minimal role to install and configure ddclient from GitHub.
Also sets up a systemd service to enable daemon-mode for ddclient.

Requirements
------------

- A recent Ansible version. This role supports the 2 most recent major Ansible releases.
  Older versions might still work, but are not supported
- A host running:
  - Ubuntu 18.04 LTS or newer
  - Debian 10 or newer
  - Other distributions may work, but are not supported (Feel free to add support with a PR!)

Role Variables
--------------

### Installation

##### `ddclient_version`
- Version of ddclient to install
- Can be a branch, tag, commit-ish or other value supported by git.
- Default: `develop`

##### `ddclient_executable_path`
- Where to put the ddclient executable
- Default is `/usr/local/sbin/ddclient`, as to not interfere with any distro-packages

##### `ddclient_configfile`
- Config file to use for the ddclient installation
- Default: `/etc/ddclient.conf`

##### `ddclient_pidfile`
- PID file to use for the ddclient daemon
- Default: `/var/run/ddclient.pid`

##### `ddclient_systemd_unit`
- Name of the unit file for the ddclient daemon
- Default: `ddclient`

### Configuration

##### `ddclient_interval`
- Number of seconds between DynDNS IP checks
- Default: `300`

##### `ddclient_mail`
- Send all updates to this user/mail address
- Default: `root`

##### `ddclient_mail_failure`
- Send all failures to this user/mail address
- Default: `root`

##### `ddclient_entries`
- List of ddclient configuration entries
- Each entry contains a list of options and a list of domains
  - Options map 1:1 to the ddclient parameters
- Example:
  ```yaml
  - options:
      protocol: cloudflare
      zone: domain.tld
      ttl: 60
      login: your-login-email
      password: APIKey
    domains:
      - domain.tld
      - my.domain.tld
  ```

### IP Lookup

Configuration options to determine how ddclient obtains the IP addresses it needs to update via DynDNS.
By default, all of these parameters are unset, which means that ddclient will use its builtin defaults.

**NOTE**: Most parameters below can be set for either IPv4 or IPv6.

##### `ddclient_strategy_usev[4/6]`
- Set the strategy to determine the IPv4/6 IP address to use for DynDNS updates
- Options include `webv[4/6], if[4/6], ip[4/6], fw, cmd`
- Default: undefined

##### `ddclient_strategy_ipv[4/6]`
- Set a static IP when using the `ip` strategy
- Default: undefined

##### `ddclient_strategy_ifv[4/6]`
- Get the IP address from a given interface when using the `if` strategy
- Default: undefined

##### `ddclient_strategy_webv[4/6]`
- Obtain IPv4 address from a web-based IP discovery service, either a known service or a custom url
- Default: undefined

##### `ddclient_strategy_webv[4/6]_skip`
- See ddclient docs
- Default: undefined

##### `ddclient_strategy_fwv[4/6]`
- Obtain IP address from device with this IP address or URL
- Default: undefined

##### `ddclient_strategy_fwv[4/6]_skip`
- Skip any IP addresses before this pattern in the text returned from the device
- Default: undefined

##### `ddclient_strategy_fw_login`
- Use this login when getting the IP from the device`
- Default: undefined

##### `ddclient_strategy_fw_password`
- Use this password when getting the IP from the device
- Default: undefined

##### `ddclient_strategy_cmdv[4/6]`
- Obtain IPv4 address from the output of this command
- Default: undefined


Example Playbook
----------------

```yaml
- hosts: all
  tasks:
    - name: Install ddclient
      include_role:
        name: maxhoesel.ddclient
      vars:
        ddclient_entries:
        - options:
            protocol: cloudflare
            zone: domain.tld
            ttl: 1 # automatic
            login: your-login-email
            password: APIKey
          domains:
            - domain.tld
            - my.domain.tld
        # Determine the IPv4 address using dyndns
        ddclient_strategy_usev4: webv4
        ddclient_strategy_webv4: dyndns
        # Get the IPv6 address from eth0
        ddclient_stragety_usev6: ifv4
        ddclient_strategy_ifv6: eth0
```

License
-------

GPL 3 or later

Author Information
------------------

Created and maintained by Max Hösel (@maxhoesel)
