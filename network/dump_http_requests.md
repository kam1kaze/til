## Wireshark UI

`ssh some.server.com 'sudo /usr/sbin/tcpdump -i eth3.2 -s0 -c 10000 -nn -w - port 3700' | wireshark -k -i -`

where
 * `-c 10000` exit after receiving 10K packets
 * `-s0` snap length, is the size of the packet to capture. `-s0` will set the size to unlimited - use this if you want to capture all the traffic. Needed if you want to pull binaries / files from network traffic
 * `port 3700` capturing filter

## CLI tshark

Run docker with the latest tshark version:
```
docker run -w /root -v /root:/root -v /var/run/docker.sock:/var/run/docker.sock --rm -ti --pid host --net host alpine:edge sh -c 'apk add --update --no-cache bash netcat-openbsd atop tcpdump nmap bind-tools docker tshark && TERM=screen exec bash'
```

### Get HTTP requests and responses for certain TCP session (steam) including headers and body

Unfortunatelly tshark cloudn't create new pcap file (lack of permissions error)
```
PCAP=capture.pcapng
touch $PCAP
```

capturing packets on `eth3.2` inerface with `3700` source and destination port
```
tshark -i eth3.2 -f "port 3700" -w $PCAP
```

Given a TCP stream number that matched the filter `-Y` 
```
tshark -r $PCAP -Y 'http contains "192.160.0.1"' -T fields -e tcp.stream
```
where
* `-Y 'http contains "192.160.0.1"'` specified filter to be applied before printing a decoded form of packets
* `-T fields -e tcp.stream` output certnain fields of the packet, TCP steam for us 


Get requests and responses combined in a single output for `%TCP_STREAM%`
```
tshark -r $PCAP -q -d tcp.port==3700,http -z follow,http,ascii,%TCP_STREAM%
```
where
* `d tcp.port==3700,http` will decode any traffic running over TCP port 3700 as HTTP
* `-z follow,http,ascii,%TCP_STREAM%` displays the contents of a HTTP session
* `-q` don't print packet information; this is useful if you're using a -z option to calculate statistics and don't want the packet information printed, just the statistics. 


### Show HTTP requests in realtime 

```
tshark -n -i eth3.2 -O http -f 'tcp port 3700' -Y 'http.request' -d tcp.port==3700,http
```
where
* `-O http` only show a detailed view of HTTP protocol
* `-f 'tcp port 3700'` capture filter, only 3700 port
* `-Y 'http.request'` output filter, only HTTP requests
* `-d tcp.port==3700,http` will decode any traffic running over TCP port 3700 as HTTP
