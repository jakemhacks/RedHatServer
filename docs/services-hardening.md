# System Services
This scan caught 32x fails relating to services.

System services results:
![Service Scan](/rhelscreenshots/services.png)

## Avahi Server
Avahi is a local networking networking tool. However, it uses an open port so if Avahi is not being used, the service should be disabled.

Using `systemctl | grep avahi` shows that we have active and running avahi-daemon.service and avahi-daemon.socket.
The nice thing about  open-scap is that it provides remediation scripts for most of the fails encountered.
I'll use the provided script to disable the avahi-daemon.

## Cron  and at Authorization
`cron` and `at` are both utilities for automating and scheduling tasks.
In order to manage access to these tools, I'm going to check for and disable any deny list then implement an allow list for both.
Allow lists offer more control and security than deny lists, since you only have to keep track who have access as opposed to giving everyone access except for those we decide to block.
1. There were 3 fails for this section:
    - Ensure that /etc/at.deny doesn't exist
    - Ensure that /etc/cron.allow exists
    - Ensure that /etc/cron.deny doesn't exist
2. I completed and verified these three fails, then set correct permissions on all cron related files and directories.

## dnsmasq
dnsmasq provides DNS services like caching and DHCP. This machine will not be a DNS server, so I will disable this as well.
`sudo dnf remove dnsmasq`

## LDAP
Lighweight Directory Access Protocol is a network directory access protocol and provides another potential attack surface.
Since this machine is not an LDAP client, I'll remove it.
`sudo dnf remove openldap-clients`

## CUPS
CUPS (Common Unix Printing System) allows a computer to act as a print server. I don't need this either.
`sudo systemctl mask --now cups.service`
*I didn't know this before, but `systemctl mask` creates a symlink from the services unit file to /dev/null. This prevents the service from being started manually or automatically. It's a more restrictive fix than simply disabling*

## OpenSSH
OpenSSH had 17 fails. While I will be using SSH on this server, there are still things I can do to secure it.
1. SSH Client alive count max and alive interval
These two settings make it so that if a client is unresponsive for a set amount of time, it is automatically disconnected.
I am setting the `ClientAliveInterval` to 60 and the `ClientAliveCountMax` to 3. This means that every 60 seconds, the server will send a keep-alive message to the client if the client is idle. After 3 of these keep-alive messages (so 180 seconds of inactivity) the server will disconnect the client.
2. Host-based Authentication
Host based authentication allows any user on an authenticated machine to access the server. I will disable this in favor of SSH keys/passwords. This keeps an attacker who has gained access to a client from having easy access to the server.
3. Disable Empty Password Login
This setting will explicitly disallow any SSH login with empty passwords.
`PermitEmptyPasswords no`
