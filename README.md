# DOCKER NGROK IMAGE

## BUILD IMAGE

```linux
git clone https://github.com/hteen/docker-ngrok.git
cd docker-ngrok
docker build -t hteen/ngrok .
```

## BUILD BINAY
docker run --rm -it -e DOMAIN="t.esile.me" -v /home/xudy/tmp/ngrok_build:/myfiles hteen/ngrok sh
因为镜像里默认的build.sh少编译了linux_i386,所以需要对镜像里build.sh做一下修改。
```
#!/bin/sh
set -e

if [ "${DOMAIN}" == "**None**" ]; then
    echo "Please set DOMAIN"
    exit 1
fi

cd ${MY_FILES}
if [ ! -f "${MY_FILES}/base.pem" ]; then
    openssl genrsa -out base.key 2048
    openssl req -new -x509 -nodes -key base.key -days 10000 -subj
"/CN=${DOMAIN}" -out base.pem
    openssl genrsa -out device.key 2048
    openssl req -new -key device.key -subj "/CN=${DOMAIN}" -out
device.csr
    openssl x509 -req -in device.csr -CA base.pem -CAkey base.key
-CAcreateserial -days 10000 -out device.crt
fi
cp -r base.pem /ngrok/assets/client/tls/ngrokroot.crt

cd /ngrok
# 如果需要编译不同平台的release server也可以在这里改
# GOOS=linux GOARCH=386 make release-release
make release-server
GOOS=linux GOARCH=386 make release-client
GOOS=linux GOARCH=amd64 make release-client
GOOS=windows GOARCH=386 make release-client
GOOS=windows GOARCH=amd64 make release-client
GOOS=darwin GOARCH=386 make release-client
GOOS=darwin GOARCH=amd64 make release-client
GOOS=linux GOARCH=arm make release-client

cp -r /ngrok/bin ${MY_FILES}/bin

echo "build ok !"
```
sh ./build.sh


## RUN
* you must mount your folder (E.g `/data/ngrok`) to container `/myfiles`
* if it is the first run, it will generate the binaries file and CA in your floder `/data/ngrok`

```linux
docker run -idt --name ngrok-server \
-v /data/ngrok:/myfiles \
-e DOMAIN='tunnel.hteen.cn' hteen/ngrok /bin/sh /server.sh
```

## nginx config

```linux
server {
     listen       80;
     server_name  tunnel.hteen.cn *.tunnel.hteen.cn;
     location / {
             proxy_redirect off;
             proxy_set_header Host $host;
             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header X-Forwarded-For
$proxy_add_x_forwarded_for;
             proxy_pass http://10.24.198.241:8082;
     }
 }
 server {
     listen       443;
     server_name  tunnel.hteen.cn *.tunnel.hteen.cn;
     location / {
             proxy_redirect off;
             proxy_set_header Host $host;
             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header X-Forwarded-For
$proxy_add_x_forwarded_for;
             proxy_pass http://10.24.198.241:4432;
     }
 }

```
