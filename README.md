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
fetch -o /root/bin/stv https://raw.githubusercontent.com/luckman212/stv/main/stv
chmod +x /root/bin/stv
exit
```

### Use stv

Using stv is easy. Just drop to a shell and type
```shell
stv [search_term]
```
You should see a list of states, with the following columns of information 

- protocol (`tcp, udp, icmp`...)
- direction (in/out)
- interface that the state was generated on
- rule id associated with the state
- state/creator ID (can be used to uniquely identify, and thus kill, a state)
- state description (`ESTABLISHED`, `FIN_WAIT` etc)
- "talkers" - a term I made up to represent the hosts involved
- gateway (only shown if it's not the default)

### Filtering

`stv` accepts a single optional parameter which can be used to filter the results to those matching a particular interface, rule ID, state type, IP address etc. Pad the search string with `|`s e.g. `|267|` for rule 267.

Since all output is standardized and each state is printed on a single line, output from `stv` is well-suited to piping into other tools such as grep or awk.

stv will print the total number of matching states at the bottom of the output.
