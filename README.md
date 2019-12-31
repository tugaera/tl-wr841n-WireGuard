# tl-wr841n-WireGuard

https://forum.openwrt.org/t/build-for-tp-link-tl-wr841n-d-all-versions/428

LEDE Reboot
LuCI with https
IPv6 and PPPoE
MiniUPnP
OpenVPN with mbed TLS
QoS
DDNS
Wget with https
Reghack (WiFi channels 12 and 13)
Minified LuCI's CSS and JS
Compiled with GCC 6.3
Removed kernel debugging
Nothing else has been touched. Basically, it is a 'make defconfig' + profile + the mentioned above
Free ROM: 76 KB
OpenVPN AES-128-CBC speed: 12Mbs/12Mbs (measured on v8 downloading ubuntu via a torrent)

https://www.dropbox.com/sh/w91lnen1g9ikuf3/AACyMjOACC0-WVFzi1Lb1O0da?dl=0

https://gist.github.com/amq/f3895b208850ab07c5e2e855fa5fcab9

```
# Configuration parameters
WG_IF="wg0"
WG_SERV="2.X.X.x"
WG_PORT="51820"
WG_ADDR="10.0.0.8/32"
# WG_ADDR6="fdf1:7610:d152:3a9c::2/64"

umask u=rw,g=,o=
wg genkey | tee wgclient.key | wg pubkey > wgclient.pub
 
# WG_KEY="$(cat wgclient.key)"
WG_KEY="+Jae="
# WG_PSK=""
WG_PUB="O+v="

# Configure firewall
uci rename firewall.@zone[0]="lan"
uci rename firewall.@zone[1]="wan"
uci rename firewall.@forwarding[0]="lan_wan"
uci del_list firewall.wan.network="${WG_IF}"
uci add_list firewall.wan.network="${WG_IF}"
uci commit firewall
/etc/init.d/firewall restart

# Configure network
uci -q delete network.${WG_IF}
uci set network.${WG_IF}="interface"
uci set network.${WG_IF}.proto="wireguard"
uci set network.${WG_IF}.private_key="${WG_KEY}"
uci set network.${WG_IF}.listen_port="${WG_PORT}"
uci add_list network.${WG_IF}.addresses="${WG_ADDR}"
# uci add_list network.${WG_IF}.addresses="${WG_ADDR6}"
 
# Add VPN peers
uci -q delete network.wgserver
uci set network.wgserver="wireguard_${WG_IF}"
uci set network.wgserver.public_key="${WG_PUB}"
# uci set network.wgserver.preshared_key="${WG_PSK}"
uci set network.wgserver.endpoint_host="${WG_SERV}"
uci set network.wgserver.endpoint_port="${WG_PORT}"
uci set network.wgserver.route_allowed_ips="1"
uci set network.wgserver.persistent_keepalive="25"
uci add_list network.wgserver.allowed_ips="0.0.0.0/1"
uci add_list network.wgserver.allowed_ips="128.0.0.0/1"
uci add_list network.wgserver.allowed_ips="::/0"
uci commit network
/etc/init.d/network restart

traceroute openwrt.org
```
