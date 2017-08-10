# check_netscaler Nagios Plugin

A Nagios Plugin written for the Citrix NetScaler Application Delivery Controller. It's based on Perl (Nagios::Plugin) and using the the NITRO REST API. No need for SNMP.

Currently the plugin has the following subcommands:

- **state:** check the current service state of vservers (e.g. lb, vpn, gslb), services and service groups
- **string, string_not:** check if a string exists in the api response or not (e.g. HA or cluster status)
- **above, below:** check if a value is above/below a threshold (e.g. traffic limits, concurrent connections)
- **sslcert:** check the lifetime for installed ssl certificates
- **nsconfig:** check for configuration changes which are not saved to disk
- **staserver:** check if configured STA servers are available
- **server:** check status of Load Balancing Servers
- **hwinfo:** just print information about the Netscaler itself
- **interfaces:** check state of all interfaces and add performance data for each interface
- **performancedata:** gather performancedata from all sorts of API endpoints
- **debug:** debug command, print all data for a endpoint

This plugin works with VPX, MPX, SDX and CPX NetScaler Appliances. The api responses may differ by build, appliance type and your installed license.

The plugin supports performance data for the commands state and the above or below threshold checks.

Feedback and feature requests are appreciated. 

# Installation

On a Enterprise Linux machine (CentOS, RHEL) execute the following commands to install all Perl dependencies (Nagios::Plugin, LWP, JSON):

    yum install perl-libwww-perl perl-JSON perl-Nagios-Plugin

If you want to connect to your NetScaler with SSL/HTTPS you should also install the LWP HTTPS package.

    yum install perl-LWP-Protocol-https

# Usage Examples

## Check status of vServers
    # NetScaler::LBvServer
    ./check_netscaler.pl -H ${IPADDR} -s -C state -o lbvserver

    # NetScaler::LBvServer::Website
    ./check_netscaler.pl -H ${IPADDR} -s -C state -o lbvserver -n vs_lb_http_webserver

    # NetScaler::VPNvServer
    ./check_netscaler.pl -H ${IPADDR} -s -C state -o vpnvserver

    # NetScaler::GSLBvServer
    ./check_netscaler.pl -H ${IPADDR} -s -C state -o gslbvserver

    # NetScaler::AAAvServer
    ./check_netscaler.pl -H ${IPADDR} -s -C state -o authenticationvserver

    # NetScaler::CSvServer
    ./check_netscaler.pl -H ${IPADDR} -s -C state -o csvserver

    # NetScaler::SSLvServer (obsolet and replaced by lbvserver for newer builds)
    ./check_netscaler.pl -H ${IPADDR} -s -C state -o sslvserver

## Check status of services
    # NetScaler::Services
    ./check_netscaler.pl -H ${IPADDR} -s -C state -o service

    # NetScaler::Services::Webserver
    ./check_netscaler.pl -H ${IPADDR} -s -C state -o service -n svc_webserver

## Check status of servicegroups
    # NetScaler::Servicegroups
    ./check_netscaler.pl -H ${IPADDR} -s -C state -o servicegroup

    # NetScaler::Servicegroups::Webservers
    ./check_netscaler.pl -H ${IPADDR} -s -C state -o servicegroup -n sg_webservers

## Check system health
    # NetScaler::Memory
    ./check_netscaler.pl -H ${IPADDR} -s -C above -o system -n memusagepcnt -w 75 -c 80

    # NetScaler::CPU
    ./check_netscaler.pl -H ${IPADDR} -s -C above -o system -n cpuusagepcnt -w 75 -c 80

    # NetScaler::CPUMGMT
    ./check_netscaler.pl -H ${IPADDR} -s -C above -o system -n mgmtcpuusagepcnt -w 75 -c 80

    # NetScaler::Disk0
    ./check_netscaler.pl -H ${IPADDR} -s -C above -o system -n disk0perusage -w 75 -c 80

    # NetScaler::Disk1
    ./check_netscaler.pl -H ${IPADDR} -s -C above -o system -n disk1perusage -w 75 -c 80

## Check high availability status
    # NetScaler::HA::Status
    ./check_netscaler.pl -H ${IPADDR} -s -C string_not -o hanode -n hacurstatus -w YES -c YES

    # NetScaler::HA::State
    ./check_netscaler.pl -H ${IPADDR} -s -C string_not -o hanode -n hacurstate -w UP -c UP

## Check expiration of installed ssl certificates
    # NetScaler::Certs
    ./check_netscaler.pl -H ${IPADDR} -s -C sslcert -w 30 -c 10

    # NetScaler::Certs::Wildcard
    ./check_netscaler.pl -H ${IPADDR} -s -C sslcert -n wildcard.example.com -w 30 -c 10

## Check for unsaved configuration changes
    # NetScaler::Config
    ./check_netscaler.pl -H ${IPADDR} -s -C nsconfig

## Check if STA servers are working

    # NetScaler::STA
    ./check_netscaler.pl -H ${IPADDR} -s -C staserver

    # NetScaler::STA::vs_vpn_gateway
    ./check_netscaler.pl -H ${IPADDR} -s -C staserver -n vs_vpn_gateway

## Check if Load Balancer server are working

    # NetScaler::Server
    ./check_netscaler.pl -H ${IPADDR} -s -C server

## Get information about the netscaler

    # NetScaler::HWInfo
    ./check_netscaler.pl -H ${IPADDR} -s -C hwinfo

## Check status of all network interfaces

    # NetScaler::Interfaces
    ./check_netscaler.pl -H ${IPADDR} -s -C interfaces

## Request performance data
##### all fields must be defined via "-n" option and be seperated with a comma

    # NetScaler::Performancedata on Cache hit/misses
    ./check_netscaler.pl -H ${IPADDR} -s -C performancedata -o ns -n cachetothits,cachetotmisses

    # NetScaler::Performancedata on tcp connections
    ./check_netscaler.pl -H ${IPADDR} -s -C performancedata -o ns -n tcpcurclientconn,tcpcurclientconnestablished,tcpcurserverconn,tcpcurserverconnestablished

    # find more object names to check out for object type "ns"
    /check_netscaler.pl -H ${IPADDR} -s -C debug -o ns

    # more interesting performance data object types
    * ns
    * cache
    * protocolhttp
    * protocolip
    * protocoltcp

## Debug command
    # Print all LB vServers (stat endpoint)
    ./check_netscaler.pl -H ${IPADDR} -s -C debug -o lbvserver

    # Print all LB vServers (config endpoint)
    ./check_netscaler.pl -H ${IPADDR} -s -C debug -o lbvserver -e config

# Configuration File
The plugin uses the Nagios::Plugin Libary, so you can use --extra-opts and seperate the login crendetials from your nagios configuration.

e.g.

```
define command {
  command_name check_netscaler_vserver
  command_line $USER5$/3rdparty/check_netscaler/check_netscaler.pl -H $HOSTADDRESS$ --extra-opts=netscaler@$USER11$/plugins.ini -C check_vserver -I '$ARG1$'
}
```

```
[netscaler]
username=nagios
password=password
```

# Contributors
- @slauger
- @macampo
- @Velociraptor85
- @bb-ricardo

# NITRO API Documentation

You will find a full documentation about the NITRO API on your NetScaler Appliance in the "Download" area.

http://NSIP/nitro-rest.tgz (where NSIP is the IP address of your NetScaler appliance).

# Tested Firmware

Tested with NetScaler 10.5, 11.0, 11.1 and 12.0. The plugin should work with all available firmware builds.
