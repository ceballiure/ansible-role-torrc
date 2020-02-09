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
- `torrc_control_port: 0` The port on which Tor will listen for local connections from Tor controller applications, as documented in control-spec.txt. If you enable the controlport, be sure to enable either hashed control password or cookie authentication, to prevent attackers from accessing it.
- `torrc_hashed_control_password: ''`
- `torrc_cookie_authentication: false`
#### Hidden service options
- `torrc_hidden_services: []` List of `{dir: '.../hidden_service', ports: ['x y:z']}` dicts to describe hidden services. "x y:z" says to redirect requests on port x to the address y:z. Once you have configured a hidden service, you can look at the contents of the file ".../hidden_service/hostname" for the address to tell people.
#### Tor relay options
- `torrc_or_ports: []`
Required: what port(s) to advertise for incoming Tor connections.
- `torrc_address: ''`
The IP address or full DNS name for incoming connections to your
relay. Leave commented out and Tor will guess.
- `torrc_outbound_bind_address: ''`
- `torrc_outbound_bind_address_exit: ''`
- `torrc_outbound_bind_address_or: ''`
If you have multiple network interfaces, you can specify one for
outgoing traffic to use.
`torrc_outbound_bind_address_exit` will be used for all exit traffic, while
`torrc_outbound_bind_address_or` will be used for all OR and Dir connections
(DNS connections ignore `torrc_outbound_bind_address`).
If you do not wish to differentiate, use `torrc_outbound_bind_address` to
specify the same address for both in a single line.
- `torrc_nickname: ''`
A handle for your relay, so people don't have to refer to it by key.
Nicknames must be between 1 and 19 characters inclusive, and must
contain only the characters [a-zA-Z0-9].
If not set, "Unnamed" will be used.
- `torrc_relay_bandwidth_rate: ''`
- `torrc_relay_bandwidth_burst: ''`
Define these to limit how much relayed traffic you will allow. Your
own traffic is still unthrottled. Note that RelayBandwidthRate must
be at least 75 kilobytes per second.
Note that units for these config options are bytes (per second), not
bits (per second), and that prefixes are binary prefixes, i.e. 2^10,
2^20, etc.
- `torrc_accounting_max: ''`
- `torrc_accounting_start: ''`
Use these to restrict the maximum traffic per day, week, or month.
Note that this threshold applies separately to sent and received bytes,
not to their sum: setting "40 GB" may allow up to 80 GB total before
hibernating.
- `torrc_contact_info: ''`
Administrative contact information for this relay or bridge. This line
can be used to contact you if your relay or bridge is misconfigured or
something else goes wrong. Note that we archive and publish all
descriptors containing these lines and that Google indexes them, so
spammers might also collect them. You may want to obscure the fact that
it's an email address and/or generate a new address for this purpose.
**If you are running multiple relays, you MUST set this option.**
You might also include your PGP or GPG fingerprint if you have one.
- `torrc_dir_ports: []`
Use this to mirror directory information for others. Please do
if you have enough bandwidth.
- `torrc_dir_port_front_page: ''`
Use to return an arbitrary blob of html on your DirPort. Now you
can explain what Tor is if anybody wonders why your IP address is
contacting them. See contrib/tor-exit-notice.html in Tor's source
distribution for a sample.
- `torrc_my_family: ''`
Fill this if you run more than one Tor relay, and add the identity
key fingerprint of each Tor relay you control, even if they're on
different networks. You declare it here so Tor clients can avoid
using more than one of your relays in a single circuit. See
https://www.torproject.org/docs/faq#MultipleRelays
However, you should never include a bridge's fingerprint here, as it would
break its concealability and potentially reveal its IP/TCP address.
**If you are running multiple relays, you MUST set this option.**
- `torrc_exit_relay: false`
- `torrc_ipv6_exit: false`
- `torrc_reduced_exit_policy: false`
- `torrc_exit_policies: []`
- `torrc_bridge_relay: false`
- `torrc_publish_server_descriptor: true`
#### Other options
- `torrc_includes: []`
Configuration options can be imported from files or folders using the %include
option with the value being a path. If the path is a file, the options from the
file will be parsed as if they were written where the %include option is. If
the path is a folder, all files on that folder will be parsed following lexical
order. Files starting with a dot are ignored. Files on subfolders are ignored.
The %include option can be used recursively.
- `torrc_raw: ''`
This option can be used to write an arbitrary torrc fragment at the end of the
template. This is useful for templating more rare torrc options not exposed
by this role.

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
    torrc_control_port: '9051'
    torrc_hashed_control_password: '16:872860B76453A77D60CA2BB8C1A7042072093276A3D701AD684053EC4C'
    torrc_cookie_authentication: true
    torrc_hidden_services:
      - dir: '@LOCALSTATEDIR@/lib/tor/hidden_service/'
        ports:
          - '80 127.0.0.1:80'
      - dir: '@LOCALSTATEDIR@/lib/tor/other_hidden_service/'
        ports:
          - '80 127.0.0.1:80'
          - '22 127.0.0.1:22'
    torrc_or_ports:
      ## If you want to listen on a port other than the one advertised in
      ## ORPort (e.g. to advertise 443 but bind to 9090), you can do it as
      ## follows.  You'll need to do ipchains or other port forwarding
      ## yourself to make this work.
      - '443 NoListen'
      - '127.0.0.1:9090 NoAdvertise'
      ## If you want to listen on IPv6 your numeric address must be explictly
      ## between square brackets as follows. You must also listen on IPv4.
      - '[2001:DB8::1]:9050'
    torrc_address: 'noname.example.com'
    torrc_outbound_bind_address: ''
    torrc_outbound_bind_address_exit: '10.0.0.4'
    torrc_outbound_bind_address_or: '10.0.0.5'
    torrc_nickname: 'ididnteditheconfig'
    # Throttle traffic to 100KB/s (800Kbps)
    torrc_relay_bandwidth_rate: '100 KBytes'
    # But allow bursts up to 200KB (1600Kb)
    torrc_relay_bandwidth_burst: '200 KBytes'
    ## Set a maximum of 40 gigabytes each way per period.
    torrc_accounting_max: '40 GBytes'
    ## Each period starts daily at midnight (AccountingMax is per day)
    torrc_accounting_start: 'day 00:00'
    ## Each period starts on the 3rd of the month at 15:00 (AccountingMax is per month)
    torrc_accounting_start: 'month 3 15:00'
    torrc_contact_info: '0xFFFFFFFF Random Person <nobody AT example dot com>'
    torrc_dir_ports:
      ## If you want to listen on a port other than the one advertised in
      ## DirPort (e.g. to advertise 80 but bind to 9091), you can do it as
      ## follows. You'll need to do ipchains or other port
      ## forwarding yourself to make this work.
      - 80 NoListen
      - 127.0.0.1:9091 NoAdvertise
    torrc_dir_port_front_page: '@CONFDIR@/tor-exit-notice.html'
    torrc_my_family: '{{ your_family_keys_fingerprints | join(",") }}'
    torrc_exit_relay: false
    torrc_ipv6_exit: false
    torrc_reduced_exit_policy: false
    torrc_exit_policies: []
    torrc_bridge_relay: false
    torrc_publish_server_descriptor: true
    torrc_includes:
      - /etc/torrc.d/
      - /etc/torrc.custom
    torrc_raw: |
      # You can add an arbitrary torrc block at the end
      # Note that this are not YAML comments but torrc comments
      # because of the YAML multiline
```

License
-------

AGPL

Author Information
------------------

Ceballiure
