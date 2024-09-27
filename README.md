# Método 01 - Raspberry Pi

- Instalar Docker Engine
- Executar o container

![](https://github.com/esfonseca/nginx-rtmp-docker-compose/blob/master/img/nginx-rtmp_OBS-RASPBERRYMODE.drawio.png)

# Método 02 - WSL2 Ubuntu

![](https://github.com/esfonseca/nginx-rtmp-docker-compose/blob/master/img/nginx-rtmp_OBS-WSL-MODE.drawio.png)

## Observações de desempenho (Método 02 - WSL2 Ubuntu)

Consumo do container com `docker stats` de 61Mb de memória e média de 13% de CPU (Core i7 13700H) para uma trasmissão 3Mbps FULL HD.

- 1 OBS no Host Windows consumindo o streaming
- 1 Smarphone com VLC consumindo o streamig

![](https://github.com/esfonseca/nginx-rtmp-docker-compose/blob/master/img/cena_exemplo_obs.gif)

## Liberar Firewall Windows

- Liberar porta TCP 1935 de Entrada

        New-NetFirewallRule -DisplayName "Allow TCP 1935" -Direction Inbound -Protocol TCP -LocalPort 1935 -Action Allow -RemoteAddress 0.0.0.0/0

- Verifica Escuta da porta `netstat -an | findstr 1935`

          TCP    0.0.0.0:1935           0.0.0.0:0              LISTENING
          TCP    [::1]:1935             [::]:0                 LISTENING

## OBS Windows de Transmissão (localhost)

- Importar `CENA_RTMP_EXEMPLO.json` para dentro do OBS

![](https://github.com/esfonseca/nginx-rtmp-docker-compose/blob/master/img/import_scene_obs.png)

## Configuração de Transmissão RTMP - DJI Fly

- Endereço RTMP colocar o IP do Docker Host

![](https://github.com/esfonseca/nginx-rtmp-docker-compose/blob/master/img/app_dji_fly.jpg)

## PortProxy

- Descobrir IP do WSL `ip addr | grep inet`

            inet 127.0.0.1/8 scope host lo
            inet 10.255.255.254/32 brd 10.255.255.254 scope global lo
            inet6 ::1/128 scope host
            -----> inet 172.28.54.51/20 brd 172.28.63.255 scope global eth0 <-------
--- 
- Comando para PortProxy da porta `RTMP 1935`
   
        netsh interface portproxy add v4tov4 listenport=1935           
        listenaddress=0.0.0.0 connectport=1935 connectaddress=172.28.54.51

netsh interface portproxy show v4tov4

    Escuta em ipv4:             Conectar-se a ipv4:

    Endereço        Porta       Endereço        Porta
    --------------- ----------  --------------- ----------
    0.0.0.0         1935        172.28.54.51    1935

## Docker engine install

Inside your WSL, add official [Docker](https://docker.com) keys and repository. The following steps is Debian based, please change to your WSL Linux distro from [official docker documentation](https://docs.docker.com/engine/install/):

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

Install docker and docker compose:

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Add your Linux user to docker group:

```bash
sudo usermod -a -G docker $(whoami)
```

Enable docker services:

```
sudo systemctl enable docker
sudo systemctl enable containerd
```

Restart WSL

```powershell
wsl --shutdown
wsl
```

Now docker should be up and running:

```bash
$ docker run --rm hello-world

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
c1ec31eb5944: Pull complete 
Digest: sha256:a26bff933ddc26d5cdf7faa98b4ae1e3ec20c4985e6f87ac0973052224d24302
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

## Supported tags and respective `Dockerfile` links

* [`latest` _(Dockerfile)_](https://github.com/tiangolo/nginx-rtmp-docker/blob/master/Dockerfile)

**Note**: Note: There are [tags for each build date](https://hub.docker.com/r/tiangolo/nginx-rtmp/tags). If you need to "pin" the Docker image version you use, you can select one of those tags. E.g. `tiangolo/nginx-rtmp:latest-2020-08-16`.

## Autostart WSL and keep in background

By default, WSL start only when called with wsl command and terminated after Windows terminal is closed... Microsoft don't offer a official way to run wsl vm as service during windows startup. To workaround this, a Task Scheduler can be created to start wsl every time you login in your Windows session and keep running in background.

In my case, i created a task with:

```
Task type: basic
Task Action:
    program: cmd.exe
    parameters: /c wsl bash -c "nohup bash -c 'while true; do sleep 8h; done &' &> /dev/null"
Task Trigger: At user logon
```

## nginx-rtmp - https://github.com/tiangolo/nginx-rtmp-docker

[**Docker**](https://www.docker.com/) image with [**Nginx**](http://nginx.org/en/) using the [**nginx-rtmp-module**](https://github.com/arut/nginx-rtmp-module) module for live multimedia (video) streaming.

## Description

This [**Docker**](https://www.docker.com/) image can be used to create an RTMP server for multimedia / video streaming using [**Nginx**](http://nginx.org/en/) and [**nginx-rtmp-module**](https://github.com/arut/nginx-rtmp-module), built from the current latest sources (Nginx 1.15.0 and nginx-rtmp-module 1.2.1).

This was inspired by other similar previous images from [dvdgiessen](https://hub.docker.com/r/dvdgiessen/nginx-rtmp-docker/), [jasonrivers](https://hub.docker.com/r/jasonrivers/nginx-rtmp/), [aevumdecessus](https://hub.docker.com/r/aevumdecessus/docker-nginx-rtmp/) and by an [OBS Studio post](https://obsproject.com/forum/resources/how-to-set-up-your-own-private-rtmp-server-using-nginx.50/).

The main purpose (and test case) to build it was to allow streaming from [**OBS Studio**](https://obsproject.com/) to different clients at the same time.

**GitHub repo**: <https://github.com/tiangolo/nginx-rtmp-docker>

**Docker Hub image**: <https://hub.docker.com/r/tiangolo/nginx-rtmp/>

## Details

## How to use

* For the simplest case, just run a container with this image:

```bash
docker run -d -p 1935:1935 --name nginx-rtmp tiangolo/nginx-rtmp
```

## How to test with OBS Studio and VLC

* Run a container with the command above


* Open [OBS Studio](https://obsproject.com/)
* Click the "Settings" button
* Go to the "Stream" section
* In "Stream Type" select "Custom Streaming Server"
* In the "URL" enter the `rtmp://<ip_of_host>/live` replacing `<ip_of_host>` with the IP of the host in which the container is running. For example: `rtmp://192.168.0.30/live`
* In the "Stream key" use a "key" that will be used later in the client URL to display that specific stream. For example: `test`
* Click the "OK" button
* In the section "Sources" click the "Add" button (`+`) and select a source (for example "Screen Capture") and configure it as you need
* Click the "Start Streaming" button


* Open a [VLC](http://www.videolan.org/vlc/index.html) player (it also works in Raspberry Pi using `omxplayer`)
* Click in the "Media" menu
* Click in "Open Network Stream"
* Enter the URL from above as `rtmp://<ip_of_host>/live/<key>` replacing `<ip_of_host>` with the IP of the host in which the container is running and `<key>` with the key you created in OBS Studio. For example: `rtmp://192.168.0.30/live/test`
* Click "Play"
* Now VLC should start playing whatever you are transmitting from OBS Studio

## Debugging

If something is not working you can check the logs of the container with:

```bash
docker logs nginx-rtmp
```

## Extending

If you need to modify the configurations you can create a file `nginx.conf` and replace the one in this image using a `Dockerfile` that is based on the image, for example:

```Dockerfile
FROM tiangolo/nginx-rtmp

COPY nginx.conf /etc/nginx/nginx.conf
```

The current `nginx.conf` contains:

```Nginx
worker_processes auto;
rtmp_auto_push on;
events {}
rtmp {
    server {
        listen 1935;
        listen [::]:1935 ipv6only=on;

        application live {
            live on;
            record off;
        }
    }
}
```

You can start from it and modify it as you need. Here's the [documentation related to `nginx-rtmp-module`](https://github.com/arut/nginx-rtmp-module/wiki/Directives).

## Technical details

* This image is built from the same base official images that most of the other official images, as Python, Node, Postgres, Nginx itself, etc. Specifically, [buildpack-deps](https://hub.docker.com/_/buildpack-deps/) which is in turn based on [debian](https://hub.docker.com/_/debian/). So, if you have any other image locally you probably have the base image layers already downloaded.

* It is built from the official sources of **Nginx** and **nginx-rtmp-module** without adding anything else. (Surprisingly, most of the available images that include **nginx-rtmp-module** are made from different sources, old versions or add several other components).

* It has a simple default configuration that should allow you to send one or more streams to it and have several clients receiving multiple copies of those streams simultaneously. (It includes `rtmp_auto_push` and an automatic number of worker processes).