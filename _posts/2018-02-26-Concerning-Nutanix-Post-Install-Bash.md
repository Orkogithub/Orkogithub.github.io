---
published: true
---
Thanks vwired!
[Nutanix post build configuration script](http://www.vwired.co.uk/2018/02/20/nutanix-post-build-configuration-script/)

```bash

#####################################################################
# Simple Bash Script for Post Cluster Foundation Build Automation
# Use # to ignore any commands that are not needed
# Script Configures both Nutanix cluster and VMware ESXi hostips
# SCP Script to /home/nutanix to a CVM
# Use chmod +y to allow execution#
# ./ to execute#
# All items in CAPS should be environment specific
#####################################################################

#Specify the variables to save editing the script
NCLI=/home/nutanix/prism/cli/ncli
DNS=x.x.x.x
NTP=x.x.x.x
PRISM_CENTRAL=x.x.x.x
PRISM_CENTRAL_USERNAME=xxxx
PRISM_CENTRAL_PASSWORD=xxxx
WEB_PROXY_ADDRESS=x.x.x.x
WEB_PROXY_PORT=x.x.x.x
NEW_PRISM_PASSWORD=xxxx
LDAP_IP=x.x.x.x
DOMAIN_NAME=xxxx
EMAIL=xxxx
SYSLOG_IP=x.x.x.x
SYSLOG_PORT=x.x.x.x
SMTP_SERVER_IP=x.x.x.x
VCENTER_NAME=x.x.x.x
VCENTER_IP=x.x.x.x
VCENTER_USERNAME=xxxx
VCENTER_PASSWORD=xxxx
VLAN_ID=x.x.x.x

#Parameters below here should not need changing if the variable has been set above
#If you want to ignore a command you can put a # at the start of the line

echo ******************* Nutanix Cluster Configuration *******************

#Specify DNS Server(s)
echo Set Name Servers
$NCLI cluster add-to-name-servers servers=$DNS

#Specify NTP Server(s)
echo Set NTP Servers
$NCLI cluster add-to-ntp-servers servers=$NTP

#Register with Prism Central
echo Register Prism Central Server
$NCLI multicluster add-to-multicluster external-ip-address-or-svm-ips=$PRISM_CENTRAL username=$PRISM_CENTRAL_USERNAME password=$PRISM_CENTRAL_PASSWORD

#Set Web Proxy
echo Set Web Proxy and whitelist Prism Central Server
$NCLI http-proxy add name=proxy_name address=$WEB_PROXY_ADDRESS port=$WEB_PROXY_PORT proxy-types=http,https

#Set Cluster Timezone
echo Setting Cluster Timezone
$NCLI cluster set-timezone timezone=Europe/London

#Change default Prism password
echo Setting Default Prism Password
$NCLI user change-password current-password=admin new-password=$NEW_PRISM_PASSWORD

#Create AD Connection
echo Connect to domain
$NCLI authconfig create-directory directory-type=ACTIVE_DIRECTORY connection-type=LDAP directory-url=ldap://$LDAP_IP:389 domain=$DOMAIN_NAME name=$DOMAIN_NAME

#Create AD role mapping for User Admin Role
echo Set Role Mapping for User Admin Role
$NCLI authconfig add-role-mapping role=ROLE_USER_ADMIN entity-type=GROUP entity-values=GROUP NAME name=NAME

#Create AD role mapping for Cluster Admin Role
echo Set Role Mapping for CLuster Admin Role
$NCLI authconfig add-role-mapping role=ROLE_CLUSTER_ADMIN entity-type=GROUP entity-values=GROUP NAME name=NAME

#Enable syslog with level warning for all components
echo Set Syslog with level warning for all components
$NCLI rsyslog-config set-status enable=false
$NCLI rsyslog-config add-server name=NAME ip-address=$SYSLOG_IP port=SYSLOG_PORT network-protocol=udp
$NCLI rsyslog-config add-module server-name=NAME module-name=cassandra level=ERROR
$NCLI rsyslog-config add-module server-name=NAME module-name=cerebro level=ERROR
$NCLI rsyslog-config add-module server-name=NAME module-name=curator level=ERROR
$NCLI rsyslog-config add-module server-name=NAME module-name=genesis level=ERROR
$NCLI rsyslog-config add-module server-name=NAME module-name=prism level=ERROR
$NCLI rsyslog-config add-module server-name=NAME module-name=stargate level=ERROR
$NCLI rsyslog-config add-module server-name=NAME module-name=zookeeper level=ERROR
$NCLI rsyslog-config set-status enable=true

#Set SMTP server details
echo Set SMTP server details
$NCLI cluster set-smtp-server address=$SMTP_SERVER_IP port=’25’ from-email-address=$EMAIL

#Enable Pulse
echo Enable Pulse
$NCLI pulse-config update enable-default-nutanix-email=true email-contacts=$EMAIL enable=true

#Register with vCenter for CRUD Operations
echo Regiser with vCenter Server
$NCLI managementserver add-name=$VCENTER_NAME url=$VCENTER_IP username=$VCENTER_USERNAME password=$VCENTER_PASSWORD type=”vmware”

#Configure Alerts Email Address
echo Configure Email Alerts
$NCLI alerts edit-alert-config enable=”true” enable-default-nutanix-email=”true” email-contacts=$EMAIL

#Configure vSwitch0 VLAN ID
for i in `hostips`; ssh -f root@$i ‘nohup sleep 60 && esxcfg-vswitch -p ALL -v VLAN_ID vSwitch0’;done'''
