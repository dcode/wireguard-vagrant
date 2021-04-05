# Wireguard Test Setup

I forked this from @rgl to get a quick Ubuntu setup for testing wireguard with Elastic Endpoint and detection-rules to address [this issue](https://github.com/elastic/detection-rules/issues/1074).
I changed the base box to [generic/ubuntu2004](https://app.vagrantup.com/generic/boxes/ubuntu2004) just
because I was more familiar with this base box.

## Instructions

Ensure you have a policy in [Kibana Fleet](https://www.elastic.co/guide/en/fleet/current/fleet-overview.html) that includes the Elastic Endpoint integration. Then
from the _Agents_ tab in Kibana Fleet, Click "Add Agent". Select your configured policy and in Step 3, copy the Kibana URL and enrollment tokens to the variables at the
top of the `Vagrantfile` in this repo.

Once that's set (assuming you have a Vagrant environment), you can run `vagrant up` and wait for it to finish. Your new boxes will show up in Fleet and start logging.

Now, you can SSH into the first box
```
vagrant ssh u1
```

Then, SSH into the second box over the Wireguard tunnel
```
ssh -i /vagrant/.vagrant/machines/u2/virtualbox/private_key 192.168.53.102
```

You should now have plenty of network data to trace traffic from one box to the other.

---
**Original README.md**
# WireGuard

Install the base [ubuntu 20.04 vagrant box](https://github.com/rgl/ubuntu-vagrant).

Launch the environment:

```bash
time vagrant up --provider=libvirt # or --provider=virtualbox
```

After the environment is up, each machine wireguard configuration will have
all the other machines as peers, e.g., `/etc/wireguard/wg0.conf` will be:

```plain
[Interface]
PrivateKey = +Ps5ijDqZxUtJgXvojG1fMsO6wL3SJixj9s5Glaud3U=
Address = 10.2.0.101/24
ListenPort = 51820

[Peer]
PublicKey = vv0x1c4a93XT0MYhHDHGsxJ2ZZq3uxHugKqj+pa83i0=
Endpoint = 192.168.53.101:51820
AllowedIPs = 10.2.0.101/32

[Peer]
PublicKey = 7S2H6RphXcDLyalL1T/b5Pxmr53137ZmccVRGdgPQDw=
Endpoint = 192.168.53.102:51820
AllowedIPs = 10.2.0.102/32
```

## References

* https://www.wireguard.com/
* [wg-quick(8)](https://git.zx2c4.com/wireguard-tools/about/src/man/wg-quick.8)
* https://wiki.archlinux.org/index.php/WireGuard
* https://wiki.debian.org/Wireguard
