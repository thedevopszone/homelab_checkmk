# Checkmk

## Links

- https://www.digitalocean.com/community/tutorials/how-to-monitor-server-health-with-checkmk-2-0-on-ubuntu-20-04?ref=dailydev#prerequisites
- 

## Prerequisites

1. One Ubuntu 20.04 server with a regular, non-root user with sudo privileges
2. One CentOS 8 server with a regular, non-root user with sudo privileges

## Step 1 — Installing Checkmk on Ubuntu

```
sudo apt update
wget https://download.checkmk.com/checkmk/2.0.0p18/check-mk-raw-2.0.0p18_0.focal_amd64.deb
sudo apt install -y ./check-mk-raw-2.0.0p18_0.focal_amd64.deb

# After the installation completes, you can now access the omd command. Try it out
sudo omd

```

The omd command can manage all Checkmk sites on your server. It can start and stop all the monitoring services at once, and you will use it to create Checkmk sites. First, however, you have to update the firewall settings to allow outside access to the default web ports


## Step 2 — Adjusting the Firewall Settings

```
sudo ufw allow Apache
sudo ufw status

```


## Step 3 — Creating a Checkmk Monitoring Site

```
sudo omd create monitoring
```

The URL address, default username, and password for accessing the monitoring interface are highlighted in this output. The site is now created, but it still needs to be started. To start the site, type:
```
sudo omd start monitoring
```


To access the Checkmk site, open http://your_ubuntu_server_ip/monitoring/

## Step 4 — Changing Your Administrative Password


First, open the Users page from the Setup menu on the left. The list will show all users that currently have access to the Checkmk site. On a fresh installation, it will list only two users. The first one, automation, is intended for use with automated tools; the second is the cmkadmin user you used to log in to the site.
Update the password, add an admin email, and make any other desired changes.



## Step 5 — Monitoring the First Host

You are now ready to monitor the first host. To accomplish this, you will first install check-mk-agent on the Ubuntu server. Then, you’ll restrict access to the monitoring data using xinetd.

The components installed with Checkmk are responsible for receiving, storing, and presenting monitoring information. They do not provide the information itself.

To gather the actual data, you will use Checkmk agent. Designed specifically for the job, a Checkmk agent can monitor all vital system components at once and report that information back to the Checkmk instance.

Installing the Agent
The first host you will monitor will be your_ubuntu_server—the server on which you have installed the Checkmk instance itself.

To begin, you must install the Checkmk agent. Packages for all major distributions, including Ubuntu, are available directly from the web interface. Open the Linux page under the Agents section from the Setup menu on the left. You will see the available agent downloads with the most popular packages under the first section labelled Packaged Agents.


The package check-mk-agent_2.0.0p18-1_all.deb is the one suited for Debian based distributions, including Ubuntu. Copy the download link for that package from the web browser and use that address to download the package

```
wget http://your_ubuntu_server_ip/monitoring/check_mk/agents/check-mk-agent_2.0.0p18-1_all.deb

sudo apt install -y ./check-mk-agent_2.0.0p18-1_all.deb

# Now verify the agent installation
check_mk_agent
```



Restricting Access to Monitoring Data Using xinetd
By default, the data from check_mk_agent is served using xinetd, a mechanism that outputs data on a specific network port upon accessing it. This means that you can access the check_mk_agent by using telnet to port 6556 (the default port for Checkmk) from any other computer on the internet unless your firewall configuration disallows it.

It is not a good security policy to publish vital information about servers to anyone on the internet. You should allow only hosts that run Checkmk and are under your supervision to access this data so that only your monitoring system can gather it.

If you have followed the initial server setup, including the steps about setting up a firewall, then access to Checkmk agent is blocked by default. It is, however, a good practice to enforce these access restrictions directly in the service configuration and not rely only on the firewall to guard it.

To restrict access to the agent data, edit the configuration file at /etc/xinetd.d/check_mk. Open the configuration file in your favorite editor. To use nano, type

```
sudo nano /etc/xinetd.d/check_mk

...
# configure the IP address(es) of your Nagios server here:
only_from      = 127.0.0.1
...
```
sudo systemctl restart xinetd


## Configuring the Host in the Checkmk Web Interface

To add a new host to monitor, go to the Hosts menu in the Setup menu on the left. From here, click Add host to the monitoring. You will be asked for some information about the host.



## Step 6 — Monitoring a Second CentOS Host

On your CentOS server, install xinetd

```
sudo dnf install -y xinetd
```

Now you can download and install the monitoring agent package needed for the CentOS server
```
sudo dnf install -y http://your_ubuntu_server_ip/monitoring/check_mk/agents/check-mk-agent-2.0.0p18-1.noarch.rpm
sudo check_mk_agent
```

```
sudo vi /etc/xinetd.d/check_mk

...
# configure the IP address(es) of your Nagios server here:
only_from      = your_ubuntu_server_ip
...
```
sudo systemctl restart xinetd

Firewall
```
sudo firewall-cmd --add-port=6556/tcp --permanent
sudo firewall-cmd --reload
```




