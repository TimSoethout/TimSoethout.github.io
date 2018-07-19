---
layout: post
category: 
tagline: 
tags: [proxy, cntlm, ntlm, corporate, bash, debian]
---
{% include JB/setup %}

[Cntlm](https://cntlm.sourceforge.net/) can be used as a forwarding proxy for an enterprise NTLM proxy on your development machine. This way programs that do not support NTLM can use Cntlm to access the outside world. It also removes the need to store your proxy credentials in your bash scripts, since only the NTLM-token is stored in your Cntlm config.

Each user using this can use his own configuration file (with hashed credentials) and a custom port or use the default `/etc/cntlm.conf` for the startup daemon.

Instal Cntlm using your packet manager: 

`sudo apt-get install cntlm`

Place cntlm.conf somewhere in your home directory and fill the UserName, PassNTLMv2 and Listen port.

```
#
# Cntlm Authentication Proxy Configuration
#
# NOTE: all values are parsed literally, do NOT escape spaces,
# do not quote. Use 0600 perms if you use plaintext password.
#
# Username: Your proxy user
Username        USERNAME
Domain          DOMAIN

# Generate Auth and PassNTLMv2 with `/usr/sbin/cntlm -v -c cntlm.conf -M "https://www.google.com"` (asks for password once for Username)
Auth            NTLMv2
PassNTLMv2      <hash>
 
# List of parent proxies to use. More proxies can be defined
# one per line in format <proxy_ip>:<proxy_port>
#
Proxy           my.corporate.proxy:8080
Proxy           my.backup.corporate.proxy:8080
 
# List addresses you do not want to pass to parent proxies
# * and ? wildcards can be used
#
NoProxy         localhost, 127.0.0.*, *.local
# Specify the port cntlm will listen on
# You can bind cntlm to specific interface by specifying
# the appropriate IP address also in format <local_ip>:<local_port>
# Cntlm listens on 127.0.0.1:3128 by default
#
# Choose your custom port here
Listen          3128
```

There is no need to put your password in the config file, since you can generate an authentication token using: `/usr/sbin/cntlm -v -c cntlm.conf -M "https://www.google.com"`.
The `-M` flag generates the token for you and then you can copy the resulting Auth and PassNTLMv2 into your config. You do need to set the parent proxy beforehand.

When you want to use the proxy, run: `/usr/sbin/cntlm -v -c cntlm.conf` or when using the startup daemon, activate your settings by reloading them with `sudo service cntlm restart`.

Your proxy is now available on `http://localhost:3128`, only accessible from localhost, and you can use it for everything to connect to the internet without supplying the cumbersome credentials.

Common shell variables to let (command line) programs use the proxy (such as mvn, git, etc) can be set in `.bashrc` for example:

```bash
export http_proxy="http://$proxy:$proxyport"
export https_proxy=$http_proxy
export ftp_proxy=$http_proxy
export rsync_proxy=$http_proxy
export no_proxy="localhost,127.0.0.1,*.local,*.intranet,169.254/16"
export ANT_OPTS="-Dhttp.proxyHost $proxy -Dhttp.proxyPort $proxyport"
export npm_config_proxy=$http_proxy
export npm_config_https_proxy=$https_proxy
export JAVA_OPTS=$ANT_OPTS
export SBT_OPTS=$JAVA_OPTS
git config --global http.proxy http://$proxy:$proxyport
```

