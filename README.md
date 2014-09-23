# ossec

#### Table of Contents

1. [Overview](#overview)
2. [Module Description - What the module does and why it is useful](#module-description)
3. [Usage - Configuration options and additional functionality](#usage)
4. [Reference - An under-the-hood peek at what the module is doing and how](#reference)
5. [Limitations - OS compatibility, etc.](#limitations)
6. [Development - Guide for contributing to the module](#development)
7. [Release notes](#release-notes)

## Overview

This module installs and configures OSSEC-HIDS client and server.
It requires concat module (https://github.com/ripienaar/puppet-concat)

## Module Description

The server is configured by installing the `ossec::server` class, and using optionally

 * `ossec::command`        : to define active/response command (like firewall-drop.sh)
 * `ossec::activeresponse` : to link rules to active/response command
 * `ossec:: email_alert`   : to receive to other email adress specific group of rules information

## Usage

### SERVER

```puppet
  node "mynode" inherits ... {
    class { 'ossec::server':
      mailserver_ip=>"mailserver.mycompany.com",
      ossec_emailto=>"nicolas.zin@mycompany.com",
    }

    ossec::command { 'firewallblock':
      command_name       => 'firewall-drop',
      command_executable => 'firewall-drop.sh',
      command_expect     => 'srcip'
    }

    ossec::activeresponse { 'blockWebattack':
      command_name => 'firewall-drop',
      ar_level     => 9,
      ar_rules_id  => [31153,31151]
    }
  }
```

### CLIENT
```puppet
  node "aclientnode" inherits ... {
    class { "ossec::client":
      ossec_server_ip => "10.10.130.66"
    }
  }
```

## Reference

### SERVER

#### class ossec::server
 * $mailserver_ip	:  smtp mail server,
 * $ossec_emailfrom (default: "ossec@${domain}") : email origin sent by ossec,
 * $ossec_emailto    => who will receive it,
 * $ossec_active_response (default: true) :  if active response should be configure on the server (beware to configure it on clients also),
 * $ossec_global_host_information_level (default: 8) : Alerting level for the events generated by the host change monitor (from 0 to 16)
 * $ossec_global_stat_level (default: 8) : Alerting level for the events generated by the statistical analysis (from 0 to 16)	
 * $ossec_email_alert_level (default: 7) : It correspond to a threshold (from 0 to 156 to sort alert send by email. Some alerts circumvent this threshold (when they have alert_email option),


#### function ossec::email_alert
 * $alert_email        : email to send to
 * $alert_group (default: false) : array of name of rules group 

Caution: no email will be send below the global $ossec_email_alert_level

About active-response mechanism, check the documentation (and extends the function maybe :-) ): http://www.ossec.net/main/manual/manual-active-responses

#### function ossec::command
 * $command_name        : human readable name for ossec::activeresponse usage
 * $command_executable  : name of the executable. Ossec comes preloaded with 'disable-account.sh','host-deny.sh','ipfw.sh','pf.sh','route-null.sh','firewall-drop.sh','ipfw_mac.sh','ossec-tweeter.sh','restart-ossec.sh'
 * $command_expect     (default: "srcip")
 * $timeout_allowed    (default: true)

#### function ossec::activeresponse
 * $command_name,
 * $ar_location     (default: "local"): it can be "local","server","defined-agent","all"
 * $ar_level        (default: 7)      : between 0 and 16
 * $ar_rules_id     (default: [])     : list of rules id
 * $ar_timeout      (default: 300)    : usually active reponse blocks for a certain amount of time.



### CLIENT
 * $ossec_server_ip => IP of the server
 * $ossec_active_response (default: true) => allows active response on this host

## Limitations

This is where you list OS compatibility, version compatibility, etc.

## Development

This module was forked from `nzin/puppet-ossec` so I could package it for Puppet Forge. There is
an [outstanding issue](https://github.com/nzin/puppet-ossec/issues/3) on `nzin/puppet-ossec` to
package it for the Forge. If this is implemented then I will tear down this fork.

## Todo

 * Tidy up formatting in this file
 * Debian/Ubuntu support to use AlienVault repo rather than locally-packaged .debs

## Release Notes

Copyright (C) 2011 Savoir-faire Linux
Author Nicolas Zin <nicolas.zin@savoirfairelinux.com>
Licence: GPL v2
