sudo nano /etc/rsyslog.d/00-filter-systemd-restarts.conf

if $programname == 'systemd' and ($msg contains 'rsyslog.service' or $msg contains 'Stopping System Logging Service' or $msg contains 'Starting System Logging Service' or $msg contains 'Started rsyslog.service' or $msg contains 'Stopped rsyslog.service') then stop

# Lisaks rsyslog'i enda teated
if $fromhost == 'ubuntusrv' and $syslogfacility-text == 'daemon' and ($msg contains '[origin software="rsyslogd"' or $msg contains 'imuxsock: Acquired UNIX socket' or $msg contains 'rsyslogd\'s userid changed') then stop

# AppArmor reload (kui see ka spammib)
if $msg contains 'apparmor="STATUS"' and $msg contains 'name="rsyslogd"' then stop





root@ubuntusrv:/var/log/remote# journalctl -u rsyslog -n 200
Dec 15 17:15:04 ubuntusrv systemd[1]: Starting rsyslog.service - System Logging Service...
Dec 15 17:15:04 ubuntusrv rsyslogd[897890]: imuxsock: Acquired UNIX socket '/run/systemd/journal/syslog' (fd 3>
Dec 15 17:15:04 ubuntusrv rsyslogd[897890]: rsyslogd's groupid changed to 104
Dec 15 17:15:04 ubuntusrv systemd[1]: Started rsyslog.service - System Logging Service.
Dec 15 17:15:04 ubuntusrv rsyslogd[897890]: rsyslogd's userid changed to 103
Dec 15 17:15:04 ubuntusrv rsyslogd[897890]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="897890" x->
Dec 15 17:16:58 ubuntusrv systemd[1]: Stopping rsyslog.service - System Logging Service...
Dec 15 17:16:58 ubuntusrv rsyslogd[897890]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="897890" x->
Dec 15 17:16:59 ubuntusrv systemd[1]: rsyslog.service: Deactivated successfully.
Dec 15 17:16:59 ubuntusrv systemd[1]: Stopped rsyslog.service - System Logging Service.
Dec 15 17:16:59 ubuntusrv systemd[1]: rsyslog.service: Consumed 2min 93ms CPU time.
Dec 15 17:16:59 ubuntusrv systemd[1]: Starting rsyslog.service - System Logging Service...
Dec 15 17:16:59 ubuntusrv rsyslogd[898504]: imuxsock: Acquired UNIX socket '/run/systemd/journal/syslog' (fd 3>
Dec 15 17:16:59 ubuntusrv rsyslogd[898504]: rsyslogd's groupid changed to 104
Dec 15 17:16:59 ubuntusrv systemd[1]: Started rsyslog.service - System Logging Service.
Dec 15 17:16:59 ubuntusrv rsyslogd[898504]: rsyslogd's userid changed to 103
Dec 15 17:16:59 ubuntusrv rsyslogd[898504]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="898504" x->
Dec 15 17:18:10 ubuntusrv systemd[1]: Stopping rsyslog.service - System Logging Service...
Dec 15 17:18:10 ubuntusrv rsyslogd[898504]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="898504" x->
Dec 15 17:18:10 ubuntusrv systemd[1]: rsyslog.service: Deactivated successfully.
Dec 15 17:18:10 ubuntusrv systemd[1]: Stopped rsyslog.service - System Logging Service.
Dec 15 17:18:10 ubuntusrv systemd[1]: rsyslog.service: Consumed 1min 18.225s CPU time.
Dec 15 17:18:10 ubuntusrv systemd[1]: Starting rsyslog.service - System Logging Service...
Dec 15 17:18:10 ubuntusrv rsyslogd[899021]: warning during parsing file /etc/rsyslog.conf, on or before line 5>
Dec 15 17:18:10 ubuntusrv systemd[1]: Started rsyslog.service - System Logging Service.
Dec 15 17:18:10 ubuntusrv rsyslogd[899021]: imuxsock: Acquired UNIX socket '/run/systemd/journal/syslog' (fd 3>
Dec 15 17:18:10 ubuntusrv rsyslogd[899021]: rsyslogd's groupid changed to 104
Dec 15 17:18:10 ubuntusrv rsyslogd[899021]: rsyslogd's userid changed to 103
Dec 15 17:18:10 ubuntusrv rsyslogd[899021]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="899021" x->
Dec 15 17:18:50 ubuntusrv systemd[1]: Stopping rsyslog.service - System Logging Service...
Dec 15 17:18:50 ubuntusrv rsyslogd[899021]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="899021" x->
Dec 15 17:18:50 ubuntusrv systemd[1]: rsyslog.service: Deactivated successfully.
Dec 15 17:18:50 ubuntusrv systemd[1]: Stopped rsyslog.service - System Logging Service.
Dec 15 17:18:50 ubuntusrv systemd[1]: rsyslog.service: Consumed 37.387s CPU time.
Dec 15 17:18:50 ubuntusrv systemd[1]: Starting rsyslog.service - System Logging Service...
Dec 15 17:18:50 ubuntusrv rsyslogd[899363]: warning during parsing file /etc/rsyslog.conf, on or before line 5>
Dec 15 17:18:50 ubuntusrv rsyslogd[899363]: imuxsock: Acquired UNIX socket '/run/systemd/journal/syslog' (fd 3>
Dec 15 17:18:50 ubuntusrv systemd[1]: Started rsyslog.service - System Logging Service.
Dec 15 17:18:50 ubuntusrv rsyslogd[899363]: rsyslogd's groupid changed to 104
Dec 15 17:18:50 ubuntusrv rsyslogd[899363]: rsyslogd's userid changed to 103
Dec 15 17:18:50 ubuntusrv rsyslogd[899363]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="899363" x->
Dec 15 17:21:21 ubuntusrv systemd[1]: Stopping rsyslog.service - System Logging Service...
Dec 15 17:21:21 ubuntusrv rsyslogd[899363]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="899363" x->
Dec 15 17:21:21 ubuntusrv systemd[1]: rsyslog.service: Deactivated successfully.
Dec 15 17:21:21 ubuntusrv systemd[1]: Stopped rsyslog.service - System Logging Service.
Dec 15 17:21:21 ubuntusrv systemd[1]: rsyslog.service: Consumed 2min 30.057s CPU time, 523.3M memory peak, 0B >
Dec 15 17:21:21 ubuntusrv systemd[1]: Starting rsyslog.service - System Logging Service...
Dec 15 17:21:21 ubuntusrv rsyslogd[900005]: warning during parsing file /etc/rsyslog.conf, on or before line 5>
Dec 15 17:21:21 ubuntusrv rsyslogd[900005]: imuxsock: Acquired UNIX socket '/run/systemd/journal/syslog' (fd 3>
Dec 15 17:21:21 ubuntusrv systemd[1]: Started rsyslog.service - System Logging Service.
Dec 15 17:21:21 ubuntusrv rsyslogd[900005]: rsyslogd's groupid changed to 104
Dec 15 17:21:21 ubuntusrv rsyslogd[900005]: rsyslogd's userid changed to 103
Dec 15 17:21:21 ubuntusrv rsyslogd[900005]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="900005" x->
Dec 15 17:30:42 ubuntusrv systemd[1]: Stopping rsyslog.service - System Logging Service...
Dec 15 17:30:42 ubuntusrv rsyslogd[900005]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="900005" x->
Dec 15 17:30:42 ubuntusrv systemd[1]: rsyslog.service: Deactivated successfully.
Dec 15 17:30:42 ubuntusrv systemd[1]: Stopped rsyslog.service - System Logging Service.
Dec 15 17:30:42 ubuntusrv systemd[1]: rsyslog.service: Consumed 10min 23.890s CPU time.
Dec 15 17:30:42 ubuntusrv systemd[1]: Starting rsyslog.service - System Logging Service...
Dec 15 17:30:42 ubuntusrv rsyslogd[902475]: warning during parsing file /etc/rsyslog.conf, on or before line 5>
Dec 15 17:30:42 ubuntusrv rsyslogd[902475]: imuxsock: Acquired UNIX socket '/run/systemd/journal/syslog' (fd 3>
Dec 15 17:30:42 ubuntusrv systemd[1]: Started rsyslog.service - System Logging Service.
Dec 15 17:30:42 ubuntusrv rsyslogd[902475]: rsyslogd's groupid changed to 104
Dec 15 17:30:42 ubuntusrv rsyslogd[902475]: rsyslogd's userid changed to 103
Dec 15 17:30:42 ubuntusrv rsyslogd[902475]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="902475" x->
Dec 15 17:32:27 ubuntusrv systemd[1]: Stopping rsyslog.service - System Logging Service...
Dec 15 17:32:27 ubuntusrv rsyslogd[902475]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="902475" x->
Dec 15 17:32:27 ubuntusrv systemd[1]: rsyslog.service: Deactivated successfully.
Dec 15 17:32:27 ubuntusrv systemd[1]: Stopped rsyslog.service - System Logging Service.
Dec 15 17:32:27 ubuntusrv systemd[1]: rsyslog.service: Consumed 1min 38.085s CPU time.
Dec 15 17:32:27 ubuntusrv systemd[1]: Starting rsyslog.service - System Logging Service...
Dec 15 17:32:27 ubuntusrv rsyslogd[903215]: imuxsock: Acquired UNIX socket '/run/systemd/journal/syslog' (fd 3>
Dec 15 17:32:27 ubuntusrv rsyslogd[903215]: rsyslogd's groupid changed to 104
Dec 15 17:32:27 ubuntusrv systemd[1]: Started rsyslog.service - System Logging Service.
Dec 15 17:32:27 ubuntusrv rsyslogd[903215]: rsyslogd's userid changed to 103
Dec 15 17:32:27 ubuntusrv rsyslogd[903215]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="903215" x->
Dec 15 17:36:58 ubuntusrv systemd[1]: Stopping rsyslog.service - System Logging Service...
Dec 15 17:36:58 ubuntusrv rsyslogd[903215]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="903215" x->
Dec 15 17:36:58 ubuntusrv systemd[1]: rsyslog.service: Deactivated successfully.
Dec 15 17:36:58 ubuntusrv systemd[1]: Stopped rsyslog.service - System Logging Service.
Dec 15 17:36:58 ubuntusrv systemd[1]: rsyslog.service: Consumed 4min 34.093s CPU time.
Dec 15 17:36:58 ubuntusrv systemd[1]: Starting rsyslog.service - System Logging Service...
Dec 15 17:36:59 ubuntusrv rsyslogd[904324]: warning during parsing file /etc/rsyslog.conf, on or before line 5>
Dec 15 17:36:59 ubuntusrv rsyslogd[904324]: imuxsock: Acquired UNIX socket '/run/systemd/journal/syslog' (fd 3>
Dec 15 17:36:59 ubuntusrv systemd[1]: Started rsyslog.service - System Logging Service.
Dec 15 17:36:59 ubuntusrv rsyslogd[904324]: rsyslogd's groupid changed to 104
Dec 15 17:36:59 ubuntusrv rsyslogd[904324]: rsyslogd's userid changed to 103
Dec 15 17:36:59 ubuntusrv rsyslogd[904324]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="904324" x->
Dec 15 17:43:01 ubuntusrv systemd[1]: Stopping rsyslog.service - System Logging Service...
Dec 15 17:43:01 ubuntusrv rsyslogd[904324]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="904324" x->
Dec 15 17:43:01 ubuntusrv systemd[1]: rsyslog.service: Deactivated successfully.
Dec 15 17:43:01 ubuntusrv systemd[1]: Stopped rsyslog.service - System Logging Service.
Dec 15 17:43:01 ubuntusrv systemd[1]: rsyslog.service: Consumed 7min 402ms CPU time.
Dec 15 17:43:01 ubuntusrv systemd[1]: Starting rsyslog.service - System Logging Service...
Dec 15 17:43:01 ubuntusrv systemd[1]: Started rsyslog.service - System Logging Service.
Dec 15 17:43:01 ubuntusrv rsyslogd[905671]: imuxsock: Acquired UNIX socket '/run/systemd/journal/syslog' (fd 3>
Dec 15 17:43:01 ubuntusrv rsyslogd[905671]: rsyslogd's groupid changed to 104
Dec 15 17:43:01 ubuntusrv rsyslogd[905671]: rsyslogd's userid changed to 103
Dec 15 17:43:01 ubuntusrv rsyslogd[905671]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="905671" x->
Dec 15 17:50:13 ubuntusrv systemd[1]: Stopping rsyslog.service - System Logging Service...
Dec 15 17:50:13 ubuntusrv rsyslogd[905671]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="905671" x->
Dec 15 17:50:13 ubuntusrv systemd[1]: rsyslog.service: Deactivated successfully.
Dec 15 17:50:13 ubuntusrv systemd[1]: Stopped rsyslog.service - System Logging Service.
Dec 15 17:50:13 ubuntusrv systemd[1]: rsyslog.service: Consumed 8min 12.671s CPU time.
Dec 15 17:50:13 ubuntusrv systemd[1]: Starting rsyslog.service - System Logging Service...
Dec 15 17:50:14 ubuntusrv systemd[1]: Started rsyslog.service - System Logging Service.
Dec 15 17:50:14 ubuntusrv rsyslogd[908003]: imuxsock: Acquired UNIX socket '/run/systemd/journal/syslog' (fd 3>
Dec 15 17:50:14 ubuntusrv rsyslogd[908003]: rsyslogd's groupid changed to 104
Dec 15 17:50:14 ubuntusrv rsyslogd[908003]: rsyslogd's userid changed to 103
Dec 15 17:50:14 ubuntusrv rsyslogd[908003]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="908003" x->
Dec 15 17:51:57 ubuntusrv systemd[1]: Stopping rsyslog.service - System Logging Service...
Dec 15 17:51:57 ubuntusrv rsyslogd[908003]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="908003" x->
Dec 15 17:51:57 ubuntusrv systemd[1]: rsyslog.service: Deactivated successfully.
Dec 15 17:51:57 ubuntusrv systemd[1]: Stopped rsyslog.service - System Logging Service.
Dec 15 17:51:57 ubuntusrv systemd[1]: rsyslog.service: Consumed 1min 41.830s CPU time.
Dec 15 17:51:57 ubuntusrv systemd[1]: Starting rsyslog.service - System Logging Service...
Dec 15 17:51:57 ubuntusrv systemd[1]: Started rsyslog.service - System Logging Service.
Dec 15 17:51:57 ubuntusrv rsyslogd[908764]: warning: ~ action is deprecated, consider using the 'stop' stateme>
Dec 15 17:51:57 ubuntusrv rsyslogd[908764]: imuxsock: Acquired UNIX socket '/run/systemd/journal/syslog' (fd 3>
Dec 15 17:51:57 ubuntusrv rsyslogd[908764]: rsyslogd's groupid changed to 104
Dec 15 17:51:57 ubuntusrv rsyslogd[908764]: rsyslogd's userid changed to 103
Dec 15 17:51:57 ubuntusrv rsyslogd[908764]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="908764" x->
Dec 15 17:52:40 ubuntusrv systemd[1]: Stopping rsyslog.service - System Logging Service...
Dec 15 17:52:40 ubuntusrv rsyslogd[908764]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="908764" x->
Dec 15 17:52:40 ubuntusrv systemd[1]: rsyslog.service: Deactivated successfully.
Dec 15 17:52:40 ubuntusrv systemd[1]: Stopped rsyslog.service - System Logging Service.
Dec 15 17:52:40 ubuntusrv systemd[1]: rsyslog.service: Consumed 38.468s CPU time.
Dec 15 17:52:40 ubuntusrv systemd[1]: Starting rsyslog.service - System Logging Service...
Dec 15 17:52:40 ubuntusrv systemd[1]: Started rsyslog.service - System Logging Service.
Dec 15 17:52:40 ubuntusrv rsyslogd[909008]: warning: ~ action is deprecated, consider using the 'stop' stateme>
Dec 15 17:52:40 ubuntusrv rsyslogd[909008]: imuxsock: Acquired UNIX socket '/run/systemd/journal/syslog' (fd 3>
Dec 15 17:52:40 ubuntusrv rsyslogd[909008]: rsyslogd's groupid changed to 104
Dec 15 17:52:40 ubuntusrv rsyslogd[909008]: rsyslogd's userid changed to 103
Dec 15 17:52:40 ubuntusrv rsyslogd[909008]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="909008" x->
Dec 15 17:53:51 ubuntusrv systemd[1]: Stopping rsyslog.service - System Logging Service...
Dec 15 17:53:51 ubuntusrv rsyslogd[909008]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="909008" x->
Dec 15 17:53:51 ubuntusrv systemd[1]: rsyslog.service: Deactivated successfully.
Dec 15 17:53:51 ubuntusrv systemd[1]: Stopped rsyslog.service - System Logging Service.
Dec 15 17:53:51 ubuntusrv systemd[1]: rsyslog.service: Consumed 1min 11.236s CPU time.
Dec 15 17:53:51 ubuntusrv systemd[1]: Starting rsyslog.service - System Logging Service...
Dec 15 17:53:51 ubuntusrv rsyslogd[909480]: imuxsock: Acquired UNIX socket '/run/systemd/journal/syslog' (fd 3>
Dec 15 17:53:51 ubuntusrv rsyslogd[909480]: rsyslogd's groupid changed to 104
Dec 15 17:53:51 ubuntusrv rsyslogd[909480]: rsyslogd's userid changed to 103
Dec 15 17:53:51 ubuntusrv rsyslogd[909480]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="909480" x->
Dec 15 17:53:51 ubuntusrv systemd[1]: Started rsyslog.service - System Logging Service.
Dec 15 17:57:40 ubuntusrv systemd[1]: Stopping rsyslog.service - System Logging Service...
Dec 15 17:57:40 ubuntusrv rsyslogd[909480]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="909480" x->
Dec 15 17:57:40 ubuntusrv systemd[1]: rsyslog.service: Deactivated successfully.
Dec 15 17:57:40 ubuntusrv systemd[1]: Stopped rsyslog.service - System Logging Service.
Dec 15 17:57:40 ubuntusrv systemd[1]: rsyslog.service: Consumed 4min 3.800s CPU time.
Dec 15 17:57:40 ubuntusrv systemd[1]: Starting rsyslog.service - System Logging Service...
Dec 15 17:57:40 ubuntusrv rsyslogd[910283]: imuxsock: Acquired UNIX socket '/run/systemd/journal/syslog' (fd 3>
Dec 15 17:57:40 ubuntusrv rsyslogd[910283]: rsyslogd's groupid changed to 104
Dec 15 17:57:40 ubuntusrv systemd[1]: Started rsyslog.service - System Logging Service.
Dec 15 17:57:40 ubuntusrv rsyslogd[910283]: rsyslogd's userid changed to 103
Dec 15 17:57:40 ubuntusrv rsyslogd[910283]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="910283" x->
Dec 15 18:07:23 ubuntusrv systemd[1]: Stopping rsyslog.service - System Logging Service...
Dec 15 18:07:23 ubuntusrv rsyslogd[910283]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="910283" x->
Dec 15 18:07:24 ubuntusrv systemd[1]: rsyslog.service: Deactivated successfully.
Dec 15 18:07:24 ubuntusrv systemd[1]: Stopped rsyslog.service - System Logging Service.
Dec 15 18:07:24 ubuntusrv systemd[1]: rsyslog.service: Consumed 10min 27.339s CPU time.
Dec 15 18:07:24 ubuntusrv systemd[1]: Starting rsyslog.service - System Logging Service...
Dec 15 18:07:24 ubuntusrv systemd[1]: Started rsyslog.service - System Logging Service.
Dec 15 18:07:24 ubuntusrv rsyslogd[911949]: imuxsock: Acquired UNIX socket '/run/systemd/journal/syslog' (fd 3>
Dec 15 18:07:24 ubuntusrv rsyslogd[911949]: rsyslogd's groupid changed to 104
Dec 15 18:07:24 ubuntusrv rsyslogd[911949]: rsyslogd's userid changed to 103
Dec 15 18:07:24 ubuntusrv rsyslogd[911949]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="911949" x->
Dec 15 18:09:00 ubuntusrv systemd[1]: Stopping rsyslog.service - System Logging Service...
Dec 15 18:09:00 ubuntusrv rsyslogd[911949]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="911949" x->
Dec 15 18:09:00 ubuntusrv systemd[1]: rsyslog.service: Deactivated successfully.
Dec 15 18:09:00 ubuntusrv systemd[1]: Stopped rsyslog.service - System Logging Service.
Dec 15 18:09:00 ubuntusrv systemd[1]: rsyslog.service: Consumed 1min 16.576s CPU time, 1.2G memory peak, 0B me>
Dec 15 18:09:00 ubuntusrv systemd[1]: Starting rsyslog.service - System Logging Service...
Dec 15 18:09:00 ubuntusrv systemd[1]: Started rsyslog.service - System Logging Service.
Dec 15 18:09:00 ubuntusrv rsyslogd[912356]: imuxsock: Acquired UNIX socket '/run/systemd/journal/syslog' (fd 3>
Dec 15 18:09:00 ubuntusrv rsyslogd[912356]: rsyslogd's groupid changed to 104
Dec 15 18:09:00 ubuntusrv rsyslogd[912356]: rsyslogd's userid changed to 103
Dec 15 18:09:00 ubuntusrv rsyslogd[912356]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="912356" x->
Dec 15 18:09:12 ubuntusrv systemd[1]: Stopping rsyslog.service - System Logging Service...
Dec 15 18:09:12 ubuntusrv rsyslogd[912356]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="912356" x->
Dec 15 18:09:12 ubuntusrv systemd[1]: rsyslog.service: Deactivated successfully.
Dec 15 18:09:12 ubuntusrv systemd[1]: Stopped rsyslog.service - System Logging Service.
Dec 15 18:09:12 ubuntusrv systemd[1]: rsyslog.service: Consumed 10.102s CPU time.
Dec 15 18:09:29 ubuntusrv systemd[1]: Starting rsyslog.service - System Logging Service...
Dec 15 18:09:30 ubuntusrv rsyslogd[912569]: imuxsock: Acquired UNIX socket '/run/systemd/journal/syslog' (fd 3>
Dec 15 18:09:30 ubuntusrv rsyslogd[912569]: rsyslogd's groupid changed to 104
Dec 15 18:09:30 ubuntusrv systemd[1]: Started rsyslog.service - System Logging Service.
Dec 15 18:09:30 ubuntusrv rsyslogd[912569]: rsyslogd's userid changed to 103
Dec 15 18:09:30 ubuntusrv rsyslogd[912569]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="912569" x->
Dec 15 18:09:42 ubuntusrv systemd[1]: Stopping rsyslog.service - System Logging Service...
Dec 15 18:09:42 ubuntusrv rsyslogd[912569]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="912569" x->
Dec 15 18:09:42 ubuntusrv systemd[1]: rsyslog.service: Deactivated successfully.
Dec 15 18:09:42 ubuntusrv systemd[1]: Stopped rsyslog.service - System Logging Service.
Dec 15 18:09:42 ubuntusrv systemd[1]: rsyslog.service: Consumed 8.336s CPU time.
Dec 15 18:09:42 ubuntusrv systemd[1]: Starting rsyslog.service - System Logging Service...
Dec 15 18:09:42 ubuntusrv rsyslogd[912631]: imuxsock: Acquired UNIX socket '/run/systemd/journal/syslog' (fd 3>
Dec 15 18:09:42 ubuntusrv systemd[1]: Started rsyslog.service - System Logging Service.
Dec 15 18:09:42 ubuntusrv rsyslogd[912631]: rsyslogd's groupid changed to 104
Dec 15 18:09:42 ubuntusrv rsyslogd[912631]: rsyslogd's userid changed to 103
Dec 15 18:09:42 ubuntusrv rsyslogd[912631]: [origin software="rsyslogd" swVersion="8.2312.0" x-pid="912631" x
