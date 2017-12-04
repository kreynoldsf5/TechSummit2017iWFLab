# Docker Build Notes

## Install Docker from script
`$ curl -fsSL get.docker.com -o get-docker.sh`

`$ sudo sh get-docker.sh`

`$ sudo usermod -aG docker $(whoami)`

### Run sherpa to access remote API
`$ docker run -d \
--name sherpa \
-v /var/run/docker.sock:/tmp/docker.sock \
-p 4550:4550 \
--restart always \
djenriquez/sherpa --allow`

### Spin up a couple containers for pool members
`$ docker run -d -P f5devcentral/f5-demo-app`

