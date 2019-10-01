# OCP 4.2 Disconnected install

This is my attempt to get an OCP 4.2 disconnected install working, put here
in the hopes that it will be useful to someone else, but be warned that it
hasn't worked for me yet.

Note that I've provided ignition files here - I ordinarily wouldn't advise
publishing these (especially the bootstrap.ign as it contains bootstrap
secrets), and they won't be useful to anyone (including myself) except for
learning what goes on. The certs are only valid for 24 hours after creation.

## Things you need

Most of the instructions that I've followed are available [here](https://blog.openshift.com/openshift-4-2-disconnected-install/).
The point of this repo is mainly for me, and to hopefully save someone
else some time typing.

 1. A CentOS machine to operate as the bastion. This requires Internet access
minimally to quay.io in order to mirror the images to the local registry.
 2. A local registry. Here, I've included a file `poc-registry.service` which
is a systemd unit (shamlessly stolen from [this repo](https://github.com/openshift-telco/openshift4x-poc) that will start up a local registry using `podman`. You'll
need to follow the instructions on the above referenced page in order to
set up the registry (setting up certs, etc).
 3. A webserver to host the ignition files, as well as the CoreOS disk image.

I've also included my dnsmasq.conf file that I use on the bastion machine to
set up DNS resolution for the private network. I've also included the requisite
`/etc/hosts` file (or you could use `address` lines in the `dnsmasq.conf` if you
wanted to). Since I built this from the default dnsmasq.conf it has an excess
of comments in it, I've included an uncommented version if you're already
familiar with dnsmasq. Obviously change the MAC addresses to suit your
environment.

## Machines that you see in this repo

* 172.16.1.1 - bastion server, hosts the registry
* 172.16.1.2 - haproxy server
* 172.16.1.10 - master1
* 172.16.1.11 - master2
* 172.16.1.13 - master3


