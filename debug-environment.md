### Special docker container

```
packages=(
  bash
  bash-completion
  util-linux
  util-linux-bash-completion
  pv
  netcat-openbsd
  htop
  atop
  tcpdump
  nmap
  bind-tools
  docker
  docker-bash-completion
  tshark
  strace
  procps
  lsof
  less
  curl
  conntrack-tools
)
docker run \
  --rm \
  -ti \
  -v /root:/root \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --pid host \
  --net host \
  --privileged \
  --cap-add=ALL \
  alpine:edge \
  sh -c "apk add --update ${packages[*]} && TERM=screen exec bash"
```
