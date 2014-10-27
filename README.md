# Midware

**_Consistent networking for Docker containers_**

Midware is a transparent shim layer that makes Docker containers come up with a standard machine interface as expected by general networked applications or the init system.  In particular, it allows Docker containers to support additional network interfaces which are fully setup *before* the application runs.  It also allows adding raw network devices to the container so that the init system can set it up according to the network configuration.

## Usage

The command format has three sections representing the Midware options, Docker run options and the command, separated by the delimiter `--`.

`midware [OPTIONS] -- [docker run OPTIONS] [-- COMMAND [args...]]`

Midware OPTIONS include:

```
-b             # Name of the host bridge
-i             # IP address of the interface in CIDR format
-c             # Name of the container interface
```

`[docker run OPTIONS]` are simply passed down to `docker run` when creating the container.  If `COMMAND` is not specified on the command line or if `Entrypoint`/`Cmd` don't exist in the image metadata, it defaults to `/sbin/init`.  Currently `--privileged' option is automatically added to be able to reach the root namespace from within the container.

## Examples

The following creates a Docker container with a preconfigured network interface named `eth0` bridged into an existing Linux bridge named `br0` on the host.  The new interface is assigned the specified IP address before the command runs.

```
# midware -b br0 -i 10.10.10.10/24 -c eth0 -- -it --net=none ubuntu -- ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
3: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether e6:7a:d1:68:4c:f1 brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.10/24 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::e47a:d1ff:fe68:4cf1/64 scope link tentative
       valid_lft forever preferred_lft forever
```

The following command creates a container with a raw network interface eth1 within the container and runs `/sbin/init` by default, which sets up the interface according to the network configuration.

```
# midware -b br0 -- -d ubuntu
```
