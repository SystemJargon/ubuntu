### quick easy local share without smb / smbd

```
mkdir /local-share
sudo chmod 766 /local-share/
touch /local-share/test.txt
```
---

### nftables / nft

note: examples of nftables and iptables have both interface enp4s0 and eth0 


flush ruleset

```
nft flush ruleset
```

add nft ruleset from file

```
nft -f ruleset.txt
```

example of typical DROP and LOG (contents of ruleset.txt)

```
add table ip filter
add chain ip filter INPUT { type filter hook input priority 0; policy drop; }
add chain ip filter FORWARD { type filter hook forward priority 0; policy accept; }
add chain ip filter OUTPUT { type filter hook output priority 0; policy accept; }
add chain ip filter LOGGING
add rule ip filter INPUT iifname "lo" ip protocol udp ip saddr 127.0.0.1 counter accept
add rule ip filter INPUT iifname "lo" ip protocol tcp ip saddr 127.0.0.1 counter accept
add rule ip filter INPUT iifname "enp4s0" ct state related,established counter accept
add rule ip filter INPUT iifname "eth0" ct state related,established counter accept
add rule ip filter INPUT iifname "lo" ip saddr 127.0.0.53 udp sport 53 counter accept
add rule ip filter INPUT counter jump LOGGING
add rule ip filter LOGGING limit rate 2/minute burst 5 packets counter log prefix "IPTables-Dropped: "
add rule ip filter LOGGING counter drop
```

List rules

```
nft list ruleset
```

---

move to nftables / nft from legacy iptables

reference: https://wiki.nftables.org/wiki-nftables/index.php/Moving_from_iptables_to_nftables

example with translations:

```
iptables-nft -A INPUT -i eth0 -p tcp -s 172.16.0.1 --dport 8080 -j ACCEPT
```


Save rules

```
iptables-nft-save
```

Also, you can just save all of your iptables rules like 

```
iptables-save > save.txt
``` 

then use the below to get the translated rules.

```
iptables-restore-translate -f save.txt
```


---

### IPTables (old-method)

Flush any input we have and create a new ruleset called LOGGING

```
sudo iptables -F INPUT
sudo iptables -N LOGGING
```

iptables.rules

```
*filter
:INPUT DROP [310:45879]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [2252:357980]
:LOGGING - [0:0]
-A INPUT -s 127.0.0.1/32 -i lo -p udp -j ACCEPT
-A INPUT -s 127.0.0.1/32 -i lo -p tcp -j ACCEPT
-A INPUT -i eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -i enp4s0 -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -s 127.0.0.53/32 -i lo -p udp -m udp --sport 53 -j ACCEPT
-A INPUT -j LOGGING
-A LOGGING -m limit --limit 2/min -j LOG --log-prefix "IPTables-Dropped: "
-A LOGGING -j DROP
COMMIT
```

output iptables rule

```
iptables-save > iptables.rules
```

Copy iptables to working ruleset

```
sudo cp iptables.rules /etc/iptables/rules.v4
```

