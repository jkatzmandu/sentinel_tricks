# Sentinel Tricks Repository
MS Azure Sentinel Shortcuts

These are various files and tricks; queries, functions, and templates for Microsoft Sentinel.

# OMS Agent (syslog) on non-x86/x64 Hardware
I've also been experimenting with building the OMS Agent on non-traditional (x86/x64) Linux systems.

I wanted an OMS Agent and syslog daemon running at home for a test Sentinel instance. All I have these days are Raspberry Pis. Note that these instructions assume some Linux/Unix familiarity. I may make the instructons a bit more "universal" and easily read at a later date.

Quick instructions for building the OMS Agent for a Raspberry Pi:
1) I stated with Raspbian on a Pi B 3+. If you're using Ubuntu you may be able to skip some of these steps because Raspbian is a hair different as it aims to be more lightweight.
2) Change /bin/sh to link to /bin/bash instead of /bin/dash. (cd /bin; rm sh; ln -s bash sh) The ./configure shell scripts expect specific POSIX behaviour which dash doesn't have.
3) Install GNU-awk (apt install gawk) and change /usr/bin/awk to link to /usr/bin/gawk. Again, ./configure scripts expect GNU-awk behaviour.
4) Follow the instructions here to use 'git clone' to get the latest codebsae to your Pi: https://github.com/microsoft/Build-OMS-Agent-for-Linux
5) In addition to the packages listed in that repository you may need to install: libpam-dev, libgss-dev, and ruby.
6) From the instructions from the "master" branch URL above create the bld-omsagent/omsagent/test/config/systest.conf file.
7) Cross fingers and run "make." I was dumb and didn't time this, but you're looking at well over an hour.
8) -- I'm still dealing with a compile issue. More coming soon!

# Sentinel Syslog Smarts
In many installations I sometimes see that events are being logged twice; once as a CEF event and another time it shows up in the "Syslog" table. There are a few ways to fix this.

Modifying the rsyslogd configuration files should solve this issue. rsyslogd (and most Linux/Unix daemons) process the configuration files in ASCII order. Therefore, 95-omsagent.conf will be processed BEFORE security-config-omsagent.conf. It is recommended to rename security-config-omsagent.conf to 01-security-config-omsagent.conf after the Linux OMS Agent installation is completed.

•	To rename this file run this command:
sudo mv /etc/rsyslog.d/security-config-omsagent.conf /etc/rsyslog.d/01-security-config-omsagent.conf

Additionally, to prevent the further ingestion and routing of the CEF or other data, add the directive “& stop” to the end of the 01-security-config-omsagent.conf.

•	Use the command:
sudo nano /etc/rsyslog.d/01-security-config-omsagent.conf

•	Restart the syslog daemon with the following command:
sudo service syslog restart
