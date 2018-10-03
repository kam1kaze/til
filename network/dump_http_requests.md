You can use desktop Wireshark UI

`ssh some.server.com 'sudo /usr/sbin/tcpdump -i eth3.2 -s0 -c 10000 -nn -w - port 3700' | wireshark -k -i -`

where
 * `-c 10000` exit after receiving 10K packets
 * `-s0` snap length, is the size of the packet to capture. `-s0` will set the size to unlimited - use this if you want to capture all the traffic. Needed if you want to pull binaries / files from network traffic
 * `port 3700` capturing filter

or you can use CLI tshark
```
docker run -w /root -v /root:/root -v /var/run/docker.sock:/var/run/docker.sock --rm -ti --pid host --net host alpine:edge sh -c 'apk add --update --no-cache bash netcat-openbsd atop tcpdump nmap bind-tools docker tshark && TERM=screen exec bash'
> touch capture.pcap
> tshark -i eth3.2 -f "port 3700" -w capture.pcap
> tshark -nr capture.pcap.cf.java -Y 'http contains "192.160.0.1"' -T fields -e tcp.stream
# Copy TCP stream number and paste to end end of the following command
> tshark -nr capture.pcap.cf.java -q -d tcp.port==3700,http -z follow,http,ascii,$$$STREAM_NUMBER$$$
```
