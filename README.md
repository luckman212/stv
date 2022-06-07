# stv

`stv` (pronounced _"Steve"_ is a **S**tate **T**able **V**iewer for pfSense. It's meant to be used at the commandline (console or ssh). `stv` uses pfctl and awk to generate its output.

### Why

[Troubleshooting firewall rules](https://docs.netgate.com/pfsense/en/latest/troubleshooting/firewall.html) can be tricky. There are some good tools included to help, like pfTop, trafshow, iftop, tcpdump and others. Still, I found myself struggling to get the output I wanted: a list of states where everything was reasonably structured, and every state occupied just a single line (so it can be further piped to grep, awk etc)

stv achieves this.

### Get stv

1. connect to pfSense using ssh or console.
2. select option 8.
3. paste the commands below to download `stv`, make it executable and put it in your path.
```
mkdir /root/bin
fetch xxx -o /root/bin/stv
chmod +x /root/bin/stv
exit
```

### Use stv

Using stv is easy. Just drop to a shell and type
```shell
stv
```
You should see a list of states, with the following columns of information 

- protocol (`tcp, udp, icmp`...)
- direction (in/out)
- interface that the state was generated on
- state/creator ID (can be used to uniquely identify, and thus kill, a state)
- state description (`ESTABLISHED`, `FIN_WAIT` etc)
- "talkers" - a term I made up to represent the hosts involved
- gateway (only shown if it's not the default)

