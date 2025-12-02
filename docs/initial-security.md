# Initial System Hardening

## RHEL Post-installation security procedures

So the first thing I did after install and system update was follow RHEL's initial security steps.

1. Start and enable firewalld
This is just to ensure that firewalld is running, which should have been taken care of during install.
`systemctl start firewalld && systemctl enable firewalld`
firewalld is the default firewall management tool that comes with RHEL and other distros in the Red Hat ecosystem. 
It is similar to ufw (uncomplicated firewall), in that it works between you, the user, and iptables or nftables, which manages kernel level firewall rules.

2. CIS Benchmark Baseline
Before any hardening tasks, I am running an initial compliance audit against the Center for Internet Security benchmark for RHEL 9. This is done using OpenSCAP, which is an open source implementation of NIST's SCAP framework. I basically runs a scan of the system and checks for compliance with specified best security practice benchmarks; which for this server, we will be using CIS's.
First, I'll install OpenSCAP:
```
sudo dnf install openscap-scanner scap-security-guide

# run the scan
sudo oscap xccdf eval --profile xccdf_org.ssgproject.content_profile_cis \
  --results-arf /tmp/initial-scan.xml \
  --report /tmp/initial-report.html \
  /usr/share/xml/scap/ssg/content/ssg-rhel9-ds.xml
```
![Initial Scan Report](/REHL-server/rhelscreenshots/initial-scan-result.png)

As you can see, we failed 197 tests and passed 171.
Our final score was 63%, and from what I remember from school; that's a failure.

Scrolling down on the report, which is save in the /tmp folder as an html file, we can see all of the specific tests that our system failed. We are then provided with information on how to fix these filed tests.

This took quiete a while to work through, several days, so I am not going to go through all of the details. I also did not go through all 197 fixes as this is just practice using SCAP and system hardening tools.
However, I will list a few that were made before moving on the the next task.

## CIS Hardening
- AIDE (Advnaced Intrusion Detection Environment) - File and Directory Integrity Utility
    creates a database of system files and uses that to verify integrity.
    configured to run weekly via cron job.
- System Cryptography Policy
    configures the system crypto policy to only use ciphers from the DEFAULT policy 
    and bans the SHA-1 algorithm.
- Disable Gnome Login User List
    Prevents user enumeration from the login screen
- Disable automount and autorun of removable media
- Configure screensaver and screenlock, prevent users from changing
- Configure PAM; password attempts, lockout time, etc. 
- Create a sudo logfile for easier tracking and auditing of sudo commands
