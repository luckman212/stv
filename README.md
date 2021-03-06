# stv (pronounced _"Steve"_)

<img src="steve.png" width="128">

`stv` is a **S**tate **T**able **V**iewer for [pfSense](https://github.com/pfsense/pfsense/). It's meant to be used at the commandline (console or ssh). **stv** leverages `pfctl` and `awk` to generate its output.

## Why

[Troubleshooting firewall rules](https://docs.netgate.com/pfsense/en/latest/troubleshooting/firewall.html) can be tricky. There are some good tools included to help, like pfTop, trafshow, iftop, tcpdump and others.

Still, I found myself struggling to get the output I wanted: a list of states where everything was reasonably structured, and every state occupied just a single line (so it can be further piped to grep, awk etc)

**stv** achieves this.

## Installing stv

1. connect to your pfSense instance using ssh or via the console.
2. select option **8**.
3. paste the commands below to download **stv** and prepare it for execution:
```
mkdir /root/bin
fetch -o /root/bin/stv https://github.com/luckman212/stv/releases/download/1.2.0/stv
chmod +x /root/bin/stv
rehash
```

## Using stv

Using **stv** is easy. Just drop to a shell and type:
```shell
stv
```
You should see a list of states, with the following columns of information:

- protocol (`tcp, udp, icmp`...)
- direction (in/out)
- interface that the state was generated on
- rule associated with the state (id & first few chars of description)
- state/creator ID (uniquely identifies a state—you can then kill it with `pfctl -k id -k <id>`)
- state description (`ESTABLISHED`, `FIN_WAIT` etc)
- "talkers" - the IPs and ports of the hosts involved
- gateway (shown only if it's not the default)

> **stv** works best on a wide (>170 columns or more) terminal.

## Getting Full Rule Descriptions from IDs

**stv** has a built in helper function to search your pf-generated ruleset and output the rule id along with whatever you've entered in pfSense as a description. This can help identify what rule is triggering a state when debugging.

Use `stv --rule <regex>` to use this function. Example:
```
# stv --rule ^110
110	let out anything from firewall host itself
# stv --rule block
134	block SIP ! whitelisted
135	block SIP ! whitelisted
```

## Filtering

**stv** accepts an optional parameter which can be used to filter the results to those matching a particular interface, rule ID, state type, IP address etc. The argument is a regex (regular expression).

### Tips

- run `stv -h` for usage help
- Pad the search string with `%`'s to make it explicit, otherwise it will be treated as a substring match.
- When entering expressions with special characters like `[]` or `.*` enclose them in single quotes to avoid shell interpretation.
- To search for a literal `[` or `]`, use `\\[` and `\\]`

### Examples

- `stv '%267%'` to output states associated with rule 267 (try `pfctl -vvsr` to list all rules along with the internal pf ruleIDs)
- `stv ESTAB` to only print active (established) states
- `stv ':4443 '` port 4443
- `stv '%igc[0-2]%'` show states related to interfaces igc0, igc1 and igc2

Since all output is standardized and each state is printed on a single line, output from **stv** is well-suited to piping into other tools such as grep or awk.

**stv** will print the total number of matching states at the bottom of the output.

Here's a sample screenshot showing NFS port 2049:

<img src="screenshot.png" width="1024">

## Continuous Monitoring

To have a "live" updating display, you can use `cmdwatch`. This is the recommended method.

1. install
```shell
pkg add https://pkg.freebsd.org/FreeBSD:12:amd64/latest/All/cmdwatch-0.2.0_2.txz
```

2. run
```shell
cmdwatch --interval=2 'stv :5201 '
```

If you prefer not to install any additional packages, you can use a simple shell loop instead:
```
while :; do clear; stv '%icmp.*tun_wg1%'; sleep 2; done
```

Press <kbd>⌃ctrl+c</kbd> to stop the loop.
