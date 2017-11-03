# switchproxy

A Docker based application to switch between avaiable proxies and caches traffic.

# Overview

````
                                               +--------+
                                     +-------> | Squid  |---------+
                                     |         +--------+         |
                                     |                            |
+-------------+    +-------+    +---------+    +--------+    +----------+
| Your laptop | -> | squid | -> | haproxy | -> | cntlm1 | -> | Internet |
+-------------+    +-------+    +---------+    +--------+    +----------+
                       |             |                            |
                +---------------+    |         +--------+         |
                | volume: cache |    +-------> | cntlm2 | --------+
                +---------------+              +--------+

````

# Configuring
It might me necessary to first connect to GUEST wifi to download the docker images with:
````
docker-compose pull
````

Run the update_proxy playbook as follows:
````
ansible-playbook update_proxy.yml --ask-sudo-pass
````

# Interfaces
Configure your applications to use the proxy using the below description.

# Terminal
Done automatically, just re-source the .profile in a running session:
````
. ~/.profile
````

# Docker
This one is a bit tricky. In Docker for Mac, on a running container, a hostname is provisioned automatomatically: docker.for.mac.localhost. This address can be used to connect from a container to the Mac, in this case going back into a container.

````
+-----------------------------+
| +--------------+    +-----+ |
| |some container| -> | lo0 | |
| +--------------+    +-----+ |
|                        |    |
|                        V    |
|                 +---------+ |
|                 | haproxy | |
|                 +---------+ |
+-----------------------------+
````

To make this work the hostfile is automatically updated by the playbook.
Now open the Docker preferences and use docker.for.mac.localhost:3128 as the value for both Web Server and Secure Web Server.

# Firefox
HTTP Proxy: localhost
Port: 3128

# Mac OS - Safari & other Mac tools (Does not work!)
Web Proxy Server: localhost : 3128
Secure Web Proxy Server: localhost : 3128

It seems the mDNSResponder need to be reloaded after a network change. This can be done with:

````
sudo killall -HUP mDNSResponder
````

View the HAProxy stats at: http://localhost:8080/stats

![dashboard](dashboard.png)

View caching statistics at: http://localhost:3128/squid-internal-mgr/info
