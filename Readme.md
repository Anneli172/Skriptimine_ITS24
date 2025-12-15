sudo nano /etc/rsyslog.d/00-filter-systemd-restarts.conf

if $programname == 'systemd' and ($msg contains 'rsyslog.service' or $msg contains 'Stopping System Logging Service' or $msg contains 'Starting System Logging Service' or $msg contains 'Started rsyslog.service' or $msg contains 'Stopped rsyslog.service') then stop

# Lisaks rsyslog'i enda teated
if $fromhost == 'ubuntusrv' and $syslogfacility-text == 'daemon' and ($msg contains '[origin software="rsyslogd"' or $msg contains 'imuxsock: Acquired UNIX socket' or $msg contains 'rsyslogd\'s userid changed') then stop

# AppArmor reload (kui see ka spammib)
if $msg contains 'apparmor="STATUS"' and $msg contains 'name="rsyslogd"' then stop
