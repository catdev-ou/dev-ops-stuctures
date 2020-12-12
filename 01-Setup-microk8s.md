# Setup microk8s

Installing microk8s on a hcloud vm is easy. Since microk8s is supporting high availibility setups since v.1.19 we have to decide wether we want this or a one node setup. Let' start with one node first. Create a host with enought storage and performance and spin it up. We'll be running several containerized applications, so a host with decent performance is required.

Catdev is using a 1 node cluster having the following Params
* CX41
* 8VCPU
* 16GB RAM
* 160GB SSD
* €19.08/mo
* Ubuntu 20.04
If we reach the linitations of this machine we can simply add a second one and double or triple the performance.
As an alternative we can rescale the vm up to 16VCPU and 32GB Ram.

When the host is ready, make it save to run with a public ip in the wild. This documentation will not cover those basics.

## Setup microk8s

To install microk8s, first update the system

````bash
sudo apt update
sudo apt dist-upgrade
sudo apt autoremove
````

Install snapd

````bash
sudo apt install snapd
````

Install microk8s

````bash
sudo snap install microk8s --classic
````

Join the group

````bash
sudo usermod -a -G microk8s $USER
sudo chown -f -R $USER ~/.kube
````

You will also need to re-enter the session for the group update to take place:

````bash
su - $USER
````

Add microk8s.kubctl to your path

````bash
echo "source <(microk8s.kubectl completion bash)" >> ~/.bashrc
source ~/.bashrc
````

Confirm microk8s is running with:

````bash
microk8s.kubectl cluster-info
````

Get more detail on current state with:

````bash
microk8s.kubectl get all --all-namespaces
````

Also microk8s is running on a public IP address, we have to to constrain public access to only those ports that are needed (port 22, 80 and 443).

````bash
# Enable port forwarding on all interfaces:
sudo iptables -P FORWARD ACCEPT
# Make this permanent across reboots:
sudo vim /etc/sysctl.conf
# Append this line to the end of the file:
net.ipv4.ip_forward=1
# and enable with:
sudo sysctl -p
# Also need to tweak the cbr0 bridge interface ufw rules slightly:
sudo ufw allow in on cbr0 && sudo ufw allow out on cbr0
# Allow routing:
sudo ufw default allow routed
# Allow ssh, http and https:
sudo ufw allow 22
sudo ufw allow 80
sudo ufw allow 443
# And enable the firewall:
sudo ufw enable
# Finally, restart Kubernetes:
sudo microk8s stop
sudo microk8s start
````

## Install Addons

Install microk8s addons

````bash
microk8s.enable dns dashboard ingress registry storage
````

* **dns** to resolves service names to IP addresses
* **dashboard** to get a web UI
* **ingress** to be able to split http traffic to different containers
* **registry** to store build images in our own private registry
* **storage** to manage and provide persistent storage capabilities to our deployments

## Setup Access to microk8s from a local machine

At this point, microk8s is running on our hetzner cloud and a public ip adress is pointing to it. Since all ports except 22,80 and 443 are blocked, a way to interact with the cluster needs to be implemented.

The tool we use is sshuttle as it provides a kind of vpn over ssh and will route all of the requests matching the 10.152.0.0/8 netmask (the netmask for microk8s) through the cluster instance.

Install sshutle

On mac

````bash
brew install sshuttle
````

On ubuntu

````bash
sudo apt install sshuttle
````

To set up the tunnel to your microk8s cluster, do:

````bash
sshuttle -r <user>@<host> 10.152.0.0/8
````

## Check that everything is up and running

First of all create a deployment of a simple nginx container

````bash
microk8s.kubectl create deployment nginx --image=nginx
# You should now be able to see your nginx by running:
microk8s.kubectl get all --all-namespaces
# But nginx needs a service so we can attach to it:
microk8s.kubectl create service clusterip nginx --tcp=80:80
````

The used service type is "clusterip". There are four types of service available:

* clusterip which assigns an IP address to your pod
* nodeport which maps a port on your pod to a port on your Kubernetes cluster IP
* loadbalancer which balances traffic across your pods
* externalname which maps an external CNAME to a pod

With the pod created and an ip exposed to the pod, the service is availibe via http://10.152.183.X
Get the IP of the service via

````bash
microk8s.kubectl get all --all-namespaces
````

The browser should show a "Welcome to nginx!"

Setup dns

To resolve the service via name, the following steps are neccesary

````bash
# Let's add an additional nameserver using resolvconf.
sudo apt-get install resolvconf
sudo vim /etc/resolvconf/resolv.conf.d/tail
# Add the following line:
nameserver 10.152.183.X
# , where 10.152.183.X is your Kubernetes DNS IP address
sudo systemctl restart resolvconf
````

Now nslookup should resolve an ip for the name nginx

````bash
nslookup nginx.default.svc.cluster.local
````

[Back to Table of Contents](README.md) [ Chapter 2 - Using microk8s](02-Using-microk8s.md)
