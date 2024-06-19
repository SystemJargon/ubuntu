### quick easy local share without smb / smbd

```
mkdir /local-share
sudo chmod 766 /local-share/
touch /local-share/test.txt
```
---

### move to nftables / nft from legacy iptables 

If you have existing iptables rulesets already configured then proceed with the below syntax

```
sudo iptables-save > iptables1.txt
sudo iptables-restore-translate -f iptables1.txt > nftables1.rules
sudo nft -f nftables1.rules
sudo nft list ruleset
```

---

However if you don't have existing iptables rules and rulesets and want to add rules into chains and with translations, below is an example

```
iptables-nft -A INPUT -i enp4s0 -p tcp -s 172.16.0.1 --dport 8080 -j ACCEPT
```

Save rules with iptables-nft-save can also work

```
iptables-nft-save
```

further reference: https://wiki.nftables.org/wiki-nftables/index.php/Moving_from_iptables_to_nftables

Why is my network interface named enpXXX instead of eth0?: https://askubuntu.com/questions/704361/why-is-my-network-interface-named-enp0s25-instead-of-eth0

---

### nftables / nft commands

note: examples of rulesets have both interface enp4s0 and wlo1  

List rules

```
nft list ruleset
```

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
add rule ip filter INPUT iifname "wlo1" ct state related,established counter accept
add rule ip filter INPUT iifname "lo" ip saddr 127.0.0.53 udp sport 53 counter accept
add rule ip filter INPUT counter jump LOGGING
add rule ip filter LOGGING limit rate 2/minute burst 5 packets counter log prefix "NFTables-Dropped: "
add rule ip filter LOGGING counter drop
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
-A INPUT -i enp4s0 -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -i wlo1 -m state --state RELATED,ESTABLISHED -j ACCEPT
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

note /etc/iptables/rules.v4 should persist across reboots

```
sudo cp iptables.rules /etc/iptables/rules.v4
```

