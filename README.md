# seafile-authentication-fail2ban

## Presentation

#### What is seafile ?

Seafile is a file hosting software system. Files are stored on a central server and can by synchronized with personal computers and mobile devices via the Seafile client. Files can also be accessed via the server's web interface. Seafile's functionality is similar to other popular services such as Dropbox and Google Drive, with the primary difference being that Seafile is free and open-source, enabling users to host their own Seafile servers without artificially imposed limits on storage space or client connections.

(Definition from wikipedia - https://en.wikipedia.org/wiki/Seafile)

#### What is fail2ban ?

Fail2ban is an intrusion prevention software framework which protects computer servers from brute-force attacks. Written in the Python programming language, it is able to run on POSIX systems that have an interface to a packet-control system or firewall installed locally, for example, iptables or TCP Wrapper.

(Definition from wikipedia - https://en.wikipedia.org/wiki/Fail2ban)

#### Why do I need to install this fail2ban's filter  ?

To protect your seafile website against brute force attemps. Each time a user/computer tries to connect and fails 3 times, a new line will be write in your seafile logs (`seahub.log`).

Fail2ban will check this log file and will ban all failed authentications with a new rule in your firewall.

## Installation

#### Copy and edit jail.local file

First copy the `jail.local` file to your folder `/etc/fail2ban`.

***WARNING: this file may override some parameters from your `jail.conf` file***

Edit `jail.local` with :
* ports used by your seafile website (e.g. `http,https`) ;
* logpath (e.g. `/home/yourusername/logs/seahub.log`) ;
* maxretry (default to 3 is equivalent to 9 real attemps in seafile, because one line is written every 3 failed authentications into seafile logs).

#### Copy seafile-auth.conf file

Then, copy the fail2ban filter `seafile-auth.conf` to `/etc/fail2ban/filter.d`.

#### Restart fail2ban

Finally, just restart fail2ban and check your firewall (iptables for me) :

```
sudo fail2ban-client reload
sudo iptables -S
```

Fail2ban will create a new chain for this jail.
So you should see these new lines :

```
...
-N fail2ban-seafile
...
-A fail2ban-seafile -j RETURN
```

## Tests

To do a simple test (but you have to be an administrator on your seafile server) go to your seafile webserver URL and try 3 authentications with a wrong password.

Actually, when you have done that, you are banned from http and https ports in iptables, thanks to fail2ban.

To check that :

on fail2ban

```
denis@myserver:~$ sudo fail2ban-client status seafile
Status for the jail: seafile
|- filter
|  |- File list:	/home/denis/logs/seahub.log
|  |- Currently failed:	0
|  `- Total failed:	1
`- action
   |- Currently banned:	1
   |  `- IP list:	1.2.3.4
   `- Total banned:	1
```

on iptables :

```
sudo iptables -S

...
-A fail2ban-seafile -s 1.2.3.4/32 -j REJECT --reject-with icmp-port-unreachable
...
```

To unban your IP address, just execute this command :

```
sudo fail2ban-client set seafile unbanip 1.2.3.4
```
