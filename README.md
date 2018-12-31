# boss-ansible-role-rsyslogd
ansible role to configure rsyslogd


* Syslog configuration for remote logservers for syslog-ng and rsyslog, both client and server - https://raymii.org/s/tutorials/Syslog_config_for_remote_logservers_for_syslog-ng_and_rsyslog_client_server.html
* https://rsyslog.readthedocs.io/en/latest/search.html?q=imptcp&check_keywords=yes&area=default
* https://rsyslog.readthedocs.io/en/latest/configuration/modules/imptcp.html?highlight=imptcp
* https://rsyslog.readthedocs.io/en/latest/examples/high_performance.html?highlight=imptcp
* https://rsyslog.readthedocs.io/en/latest/concepts/multi_ruleset.html?highlight=imptcp
* https://rsyslog.readthedocs.io/en/latest/configuration/modules/idx_input.html?highlight=imptcp
* https://manpages.ubuntu.com/manpages/xenial/man5/rsyslog.conf.5.html
* https://manpages.ubuntu.com/manpages/xenial/man8/rsyslogd.8.html

# rsyslog.conf

```
# Config generated by Chef - manual edits will be overwritten
#
#  /etc/rsyslog.conf  Configuration file for rsyslog.
#
#     For more information see
#     /usr/share/doc/rsyslog-doc/html/rsyslog_conf.html
#
#  Default logging rules can be found in /etc/rsyslog.d/50-default.conf
#
# Set hostname
#
#
# Set max message size
#
$MaxMessageSize 2k

#
# Preserve FQDN
#
$PreserveFQDN off

#################
#### MODULES ####
#################

$ModLoad imuxsock
$ModLoad imklog

  $ModLoad imtcp
  $InputTCPServerRun 514

###########################
#### GLOBAL DIRECTIVES ####
###########################

#
# Use traditional timestamp format.
# To enable high precision timestamps, comment out the following line.
#
$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat

# Filter duplicated messages
$RepeatedMsgReduction on

#
# Set temporary directory to buffer syslog queue
#
$WorkDirectory /var/spool/rsyslog

#
# Set the default permissions for all log files.
#
$Umask 0022
$FileOwner syslog
$FileGroup adm
$FileCreateMode 0640
$DirOwner root
$DirGroup adm
$DirCreateMode 0755

$PrivDropToUser syslog
$PrivDropToGroup syslog

#
# Set other directives
#

#
# Include all config files in /etc/rsyslog.d/
#
$IncludeConfig /etc/rsyslog.d/*.conf
```



## 35-server-per-host.conf

```
root@dokken:/etc/rsyslog.d# cat 35-server-per-host.conf
# Generated by Chef
# Local modifications will be overwritten

$DirGroup adm
$DirCreateMode 0755
$FileGroup adm

$template PerHostAuth,"/srv/rsyslog/%$YEAR%/%$MONTH%/%$DAY%/%HOSTNAME%/auth.log"
$template PerHostCron,"/srv/rsyslog/%$YEAR%/%$MONTH%/%$DAY%/%HOSTNAME%/cron.log"
$template PerHostSyslog,"/srv/rsyslog/%$YEAR%/%$MONTH%/%$DAY%/%HOSTNAME%/syslog"
$template PerHostDaemon,"/srv/rsyslog/%$YEAR%/%$MONTH%/%$DAY%/%HOSTNAME%/daemon.log"
$template PerHostKern,"/srv/rsyslog/%$YEAR%/%$MONTH%/%$DAY%/%HOSTNAME%/kern.log"
$template PerHostLpr,"/srv/rsyslog/%$YEAR%/%$MONTH%/%$DAY%/%HOSTNAME%/lpr.log"
$template PerHostUser,"/srv/rsyslog/%$YEAR%/%$MONTH%/%$DAY%/%HOSTNAME%/user.log"
$template PerHostMail,"/srv/rsyslog/%$YEAR%/%$MONTH%/%$DAY%/%HOSTNAME%/mail.log"
$template PerHostMailInfo,"/srv/rsyslog/%$YEAR%/%$MONTH%/%$DAY%/%HOSTNAME%/mail.info"
$template PerHostMailWarn,"/srv/rsyslog/%$YEAR%/%$MONTH%/%$DAY%/%HOSTNAME%/mail.warn"
$template PerHostMailErr,"/srv/rsyslog/%$YEAR%/%$MONTH%/%$DAY%/%HOSTNAME%/mail.err"
$template PerHostNewsCrit,"/srv/rsyslog/%$YEAR%/%$MONTH%/%$DAY%/%HOSTNAME%/news.crit"
$template PerHostNewsErr,"/srv/rsyslog/%$YEAR%/%$MONTH%/%$DAY%/%HOSTNAME%/news.err"
$template PerHostNewsNotice,"/srv/rsyslog/%$YEAR%/%$MONTH%/%$DAY%/%HOSTNAME%/news.notice"
$template PerHostDebug,"/srv/rsyslog/%$YEAR%/%$MONTH%/%$DAY%/%HOSTNAME%/debug"
$template PerHostMessages,"/srv/rsyslog/%$YEAR%/%$MONTH%/%$DAY%/%HOSTNAME%/messages"

auth,authpriv.*         ?PerHostAuth
*.*;auth,authpriv.none  -?PerHostSyslog
cron.*                  ?PerHostCron
daemon.*                -?PerHostDaemon
kern.*                  -?PerHostKern
lpr.*                   -?PerHostLpr
mail.*                  -?PerHostMail
user.*                  -?PerHostUser

mail.info               -?PerHostMailInfo
mail.warn               ?PerHostMailWarn
mail.err                ?PerHostMailErr

news.crit               ?PerHostNewsCrit
news.err                ?PerHostNewsErr
news.notice             -?PerHostNewsNotice

*.=debug;\
  auth,authpriv.none;\
  news.none;mail.none   -?PerHostDebug

*.=info;*.=notice;*.=warn;\
  auth,authpriv.none;\
  cron,daemon.none;\
  mail,news.none        -?PerHostMessages


#
# Stop processing of all non-local messages. You can process remote messages
# on levels less than 35.
#
:fromhost-ip,!isequal,"127.0.0.1" stop
root@dokken:/etc/rsyslog.d#
```


## 50-default.conf

```
root@dokken:/etc/rsyslog.d# cat 50-default.conf
# Generated by Chef
# For more information see rsyslog.conf(5) and /etc/rsyslog.conf

auth,authpriv.*    /var/log/auth.log
*.*;auth,authpriv.none    -/var/log/syslog
daemon.*    -/var/log/daemon.log
kern.*    -/var/log/kern.log
mail.*    -/var/log/mail.log
user.*    -/var/log/user.log
mail.info    -/var/log/mail.info
mail.warn    -/var/log/mail.warn
mail.err    /var/log/mail.err
news.crit    /var/log/news/news.crit
news.err    /var/log/news/news.err
news.notice    -/var/log/news/news.notice
*.=debug;auth,authpriv.none;news.none;mail.none    -/var/log/debug
*.=info;*.=notice;*.=warn;auth,authpriv.none;cron,daemon.none;mail,news.none    -/var/log/messages
*.emerg    :omusrmsg:*
root@dokken:/etc/rsyslog.d#
```


# rsyslog import modules

https://rsyslog.readthedocs.io/en/latest/configuration/modules/idx_input.html?highlight=imptcp

```
im3195: RFC3195 Input Module
imfile: Text File Input Module
imgssapi: GSSAPI Syslog Input Module
imjournal: Systemd Journal Input Module
imklog: Kernel Log Input Module
imkmsg: /dev/kmsg Log Input Module
impstats: Generate Periodic Statistics of Internal Counters
imptcp: Plain TCP Syslog
imrelp: RELP Input Module
imsolaris: Solaris Input Module
imtcp: TCP Syslog Input Module
imudp: UDP Syslog Input Module
imuxsock: Unix Socket Input
```


```
# - comment: 'Log messages in journald using module imjournal'
#   options: |-
#     $ModLoad imjournal
#     $OmitLocalLogging off
#     $SystemLogSocketName /run/systemd/journal/syslog
#
```
