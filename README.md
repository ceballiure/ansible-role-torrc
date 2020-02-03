Torrc
=========

Templae a torrc configuration file. And nothing else!

Based on [configuration file for a typical Tor user](https://FIXME), last updated 28 February 2019 for Tor 0.3.5.1-alpha (may or may not work for much older or much newer versions of Tor.)

This role tries to honor the default variables of a tor daemon.

See [the tor manual](https://www.torproject.org/docs/tor-manual.html) for more options you can use in this file.

Role Variables
--------------

### Ansible-specific variables

- `torrc_owner: tor`
- `torrc_group: tor`
- `torrc_dest: /etc/tor/torrc` the folder should already exist
- `torrc_notify: restart_tor_service` state your handler name

### Tor-specific variables

- `torrc_socks_ports: ['9050']` Tor opens a SOCKS proxy on port 9050 by default -- even if you don't configure one. Set `torrc_socks_ports: ['0']` if you plan to run Tor only as a relay, and not make any local application connections yourself.
- `torrc_socks_policies: []` Entry policies to allow/deny SOCKS requests based on IP address. First entry that matches wins. If no SOCKSPolicy is set, we accept all (and only) requests that reach a SOCKSPort. Untrusted users who can access your SOCKSPort may be able to learn about the connections you make.
- `torrc_logs: []` Logs go to stdout at level "notice" unless redirected by something else, like one of the example lines. You can have as many Log lines as you want.  We advise using "notice" in most cases, since anything more verbose may provide sensitive information to an attacker who obtains the logs.
- `torrc_run_as_daemon: false` Set to `true` to start the process in the background.
- `torrc_data_directory: ''` The directory for keeping all the keys/etc. By default, we store things in $HOME/.tor on Unix.
- `torrc_control_port: 0`
- `torrc_hashed_control_password: ''`
- `torrc_cookie_authentication: false`
#### Hidden service options
- `torrc_hidden_services: []`
#### Tor relay options
- `torrc_or_ports: []`
- `torrc_address: ''`
- `torrc_outbound_bind_address: ''`
- `torrc_outbound_bind_address_exit: ''`
- `torrc_outbound_bind_address_or: ''`
- `torrc_nickname: ''`
- `torrc_relay_bandwidth_rate: ''`
- `torrc_relay_bandwidth_burst: ''`
- `torrc_accounting_max: ''`
- `torrc_accounting_start: ''`
- `torrc_contact_info: ''`
- `torrc_dir_ports: []`
- `torrc_dir_port_front_page: ''`
- `torrc_my_family: ''`
- `torrc_exit_relay: false`
- `torrc_ipv6_exit: false`
- `torrc_reduced_exit_policy: false`
- `torrc_exit_policies: []`
- `torrc_bridge_relay: false`
- `torrc_publish_server_descriptor: true`
#### Other options
- `torrc_includes: []`
- `torrc_??????: ''` custom torrc blob


Example Playbook
----------------

```
- hosts: tor
  roles:
    - ceballiure.torrc
  vars:
    torrc_socks_ports:
      - '9050'  # Default: Bind to localhost:9050 for local connections.
      - '192.168.0.1:9100'  # Bind to this address:port too.
    torrc_socks_policies:
      - accept 192.168.0.0/16
      - accept6 FC00::/7
      - reject *
    torrc_logs:
      # Send all messages of level 'notice' or higher to @LOCALSTATEDIR@/log/tor/notices.log
      - notice file @LOCALSTATEDIR@/log/tor/notices.log
      # Send every possible message to @LOCALSTATEDIR@/log/tor/debug.log
      - debug file @LOCALSTATEDIR@/log/tor/debug.log
      # Use the system log instead of Tor's logfiles
      - notice syslog
      # To send all messages to stderr
      - debug stderr
    torrc_data_directory: '@LOCALSTATEDIR@/lib/tor'
    

```

License
-------

AGPL

Author Information
------------------

Ceballiure
