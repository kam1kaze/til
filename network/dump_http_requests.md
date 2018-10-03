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
tshark -r $PCAP -Y 'http contains "194.200.100.10"' -T fields -e tcp.stream
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

Example output:
```
# tshark -r $PCAP -q -d tcp.port==3700,http -z follow,http,ascii,61
===================================================================
Follow: http,ascii
Filter: tcp.stream eq 61
Node 0: 10.6.1.21:33384
Node 1: 10.6.1.23:3700
409
GET /ws HTTP/1.1
Host: test.example.com
X-Real-IP: 194.200.100.10
X-Forwarded-For: 194.200.100.10, 194.20.10.1
X-Forwarded-Proto: https
Upgrade: websocket
Connection: upgrade
Accept-Encoding: gzip
CF-IPCountry: US
CF-RAY: c7c463f309e48dec-VIE
CF-Visitor: {"scheme":"https"}
Sec-WebSocket-Version: 13
Sec-WebSocket-Key: VoBw929XRwjmwe2hvJd2XX==
CF-Connecting-IP: 194.20.10.1


        299
HTTP/1.1 101 Switching Protocols
Date: Wed, 03 Oct 2018 11:59:18 GMT
Access-Control-Allow-Origin: *
Content-Encoding: gzip
Content-Type: application/json;charset=utf-8
Connection: Upgrade
Sec-WebSocket-Accept: kYeAmFJVvwW0tpkvLFkKgKLbZdU=
Server: Jetty(9.4.z-SNAPSHOT)
Upgrade: WebSocket


===================================================================
```


### Show HTTP requests in realtime 

```
tshark -n -i eth3.2 -O http -f 'tcp port 666 and http contains portquiz.net' -Y 'http.request' -d tcp.port==666,http
```
where
* `-O http` only show a detailed view of HTTP protocol
* `-f 'tcp port 3700'` capture filter, only 3700 port
* `-Y 'http.request'` output filter, only HTTP requests
* `-d tcp.port==3700,http` will decode any traffic running over TCP port 3700 as HTTP

Example output:
```
[terminal 1]# curl -v http://portquiz.net:666

[terminal 2]# tshark -n -i eth3.2 -O http -f 'tcp port 666' -Y 'http.request' -d tcp.port==666,http
Capturing on 'eth3.2'
Frame 4: 146 bytes on wire (1168 bits), 146 bytes captured (1168 bits) on interface 0
Ethernet II, Src: 0c:c4:7a:32:b6:1v, Dst: c2:2a:10:7f:9a:c0
Internet Protocol Version 4, Src: 10.6.1.21, Dst: 5.196.70.86
Transmission Control Protocol, Src Port: 45526, Dst Port: 666, Seq: 1, Ack: 1, Len: 80
Hypertext Transfer Protocol
    GET / HTTP/1.1\r\n
        [Expert Info (Chat/Sequence): GET / HTTP/1.1\r\n]
            [GET / HTTP/1.1\r\n]
            [Severity level: Chat]
            [Group: Sequence]
        Request Method: GET
        Request URI: /
        Request Version: HTTP/1.1
    User-Agent: curl/7.29.0\r\n
    Host: portquiz.net:666\r\n
    Accept: */*\r\n
    \r\n
    [Full request URI: http://portquiz.net:666/]
    [HTTP request 1/1]
```
