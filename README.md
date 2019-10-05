# OCP 4.2 Disconnected install

This is my attempt to get an OCP 4.2 disconnected install working, put here
in the hopes that it will be useful to someone else. This has led to a what
I'll call semi-successful install. Why only semi-successful? The monitoring
operator is currently failing on the cluster installed by this for unknown
reasons, and the openshift-samples operator cannot function for obvious reasons.

Note that I've provided ignition files here - I ordinarily wouldn't advise
publishing these (especially the bootstrap.ign as it contains bootstrap
secrets), and they won't be useful to anyone (including myself) except for
learning what goes on. The certs are only valid for 24 hours after creation.
The cluster should be up for 24 hours after creation in order to allow time
for certificate rotation. After that point, you can shut down the cluster if you're
not using it.

## Things you need

Most of the instructions that I've followed are available [here](https://blog.openshift.com/openshift-4-2-disconnected-install/).
The point of this repo is mainly for me, and to hopefully save someone
else some time typing.

 1. A CentOS machine to operate as the bastion. This requires Internet access
minimally to quay.io in order to mirror the images to the local registry.
 2. A local registry. Here, I've included a file `poc-registry.service` which
is a systemd unit (shamlessly stolen from [this repo](https://github.com/openshift-telco/openshift4x-poc)) that will start up a local registry using `podman`. You'll
need to follow the instructions on the above referenced page in order to
set up the registry (setting up certs, etc).
 3. A webserver to host the ignition files, as well as the CoreOS disk image.
 4. A fairly beefy machine. You're running 6 (at times 7) VM's, some of which
are fairly heavyweight. I ran out of memory at one point on a Windows 10
Hyper-V machine with 64GB of RAM.

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
* 172.16.1.14 - worker1

## Things I've learned along the way

* You have to have ALL of the networking requirements taken care of prior to
starting the install. You can find the UPI installation documentation with
requirements [here](https://docs.openshift.com/container-platform/4.1/installing/installing_bare_metal/installing-bare-metal.html#installation-network-user-infra_installing-bare-metal)
* DHCP must provide hostnames, and a default route. This last part led me
to banging my head against a wall forever until I looked into the NetorkMangler
source code. It won't use a DHCP-provided hostname if there is no default route.
* Time is important. Since my virtualization environment was on my Windows 10
workstation, I used Hyper-V. Hyper-V can be somewhat stupid in how it provides
time to the firmware of the guests. I found that it was providing the host time,
in US/Eastern, to the guest which thought that the time was in UTC. This led
to not being able to retrieve ignition configs because the cert was not yet valid
since the bootstrap machine got the correct time.
* If you want to change a configuration of something that is managed by an operator
(for example having only one ingress router since I only have one worker node) then
you cannot modify the resources directly, or the operator will come in behind you
and reconcile to the "correct" configuration. You have to edit the CRD that the
operator is using. You can get a list of CRD's with `oc get crd`.
