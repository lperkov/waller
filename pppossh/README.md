This package will add the so-called `pppossh` protocol support to OpenWrt.  The idea is mainly from [`pvpn` project](https://github.com/halhen/pvpn) (poor man's VPN over SSH).

PPPoSSH is generally not considered a network setup for production use mainly due to the TCP-over-TCP styles of work, but I have found it quite handy for personal use.  And with what's already in OpenWrt, it's deadly easy to setup.

## Prerequisites and dependency.

`pppossh` depends on either `dropbear` or `openssh-client`; `dropbear` is normally enabled in OpenWrt by default.

The following requirements need to be fulfilled for it to work.

- A SSH account on the remote machine with `CAP_NET_ADMIN` capability is required.
- Public key authentication must be enabled and setup properly.

	Public key of the one generated automatially by dropbear can be induced by the following command.  But you can always use your own (dropbear can work with OpenSSH public key).

		dropbearkey -y -f /etc/dropbear/dropbear_rsa_host_key

- SSH server's fingerprint has to be present in `~/.ssh/known_hosts` for the authentication to proceed in an unattended way.

	Logging in at least once to the remote server from OpenWrt should do this for you.

## How to use it.

The protocol name to use in `/etc/config/network` is `pppossh`.  Options are as described below.

- `server`, SSH server name (*required*).
- `port`, SSH server port (defaults to `22`).
- `sshuser`, SSH login username (*required*).
- `identity`, list of client private key files.  `~/.ssh/id_{rsa,dsa}` will
   be used if no identity file was specified and at least one of them must be
   valid for the public key authentication to proceed.
- `ipaddr`, local ip address to be assigned.
- `peeraddr`, peer ip address to be assigned.
- `ssh_options`, extra options for the ssh client.

## Tips

An `uci batch` command template for your reference.  Modify it to suite your situation.

	uci batch <<EOF
	delete network.fs
	set network.fs=interface
	set network.fs.proto=pppossh
	set network.fs.sshuser=root
	set network.fs.server=ssh.example.cn
	set network.fs.port=30244
	add_list network.fs.identity=/etc/dropbear/dropbear_rsa_host_key
	set network.fs.ipaddr=192.168.177.2
	set network.fs.peeraddr=192.168.177.1
	commit
	EOF

Allow forward and NAT on the remote side (`ppp0` is the peer interface on the remote side.  `eth0` is the interface for Internet access).

	sysctl -wnet.ipv4.ip_forward=1 
	iptables -t filter -A FORWARD -i ppp0 -j ACCEPT
	iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

