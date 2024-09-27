# Windows 11 with WSL2 dev environment

Steps to install an WSL2 env with:

- wsl2 debian vm
- vscode
- docker host natively inside WSL2 (no Docker Desktop or Rancher Desktop Needed)
- TCP port forwarding in windows host using 0.0.0.0 IP instead of localhost only

## WSL2 install and config

Create a **.wslconfig** file inside your Windows User directory (Eg. _C:\Windows\Users\\**myuser**\\.wslconfig_) with following content:

```
[wsl2]
memory=4GB
```

Install WSL with your desired Linux Distribution (-d Ubuntu, Debian, etc...):

```powershell
wsl --install -d Debian
```

After WSL install, reboot the system and enable systemd inside WSL linux vm:

```powershell
wsl
```

```bash
echo '[boot]
systemd = true
' > /etc/wsl.conf

exit
```
Reboot WSL system:

```powershell
wsl --shutdown
```

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

## Install DevOps common tools inside WSL

Suggested tools... Please change it as needed.

```bash
sudo apt update
sudo apt install ansible git sshpass sshfs wget curl
```

## Visual Studio Code integrations

Inside WSL, call VScode from command line to trigger vscode-server install... That way, all code commands are automaticaly integrated with VScode remote explorer of your Windows VScode.

```bash
mkdir myproject
cd myproject
code .
```

Vscode-server will be downloaded with wget and installed automatically. Other WSL instegrations will be suggested by Windows VScode, like docker, WSL, etc... Install it as you need.

## WSL port forwardings

WSL2 on Windows do all port forwardings altomatilly from Linux to Windows Host, but only for localhost IP address (127.0.0.1...). To open your needed ports on your default LAN ip address, i recommend to use the powershell script from [John Wright Stanly's blog](https://jwstanly.com/blog/article/Port+Forwarding+WSL+2+to+Your+LAN/).

Just one adjust was needed for my Windows 11 machine to run the script correctly. I needed to change the value of "$fireWallDisplayName" variable to something without special caracthers (in my case, spaces..). Eg:

```
$fireWallDisplayName = 'WSL Port Forwarding';
```
to 

```
$fireWallDisplayName = WSL;
```

Now, just save adjusted script and exec it with administrator elevated Windows Terminal:

```powershell
powershell.exe -File "Bridge-WslPorts.ps1"
```

In my case, i need just ports 80 and 443 open since i use [Traefik Reverse Proxy](https://traefik.io/traefik/) containerized inside Linux WSL. that way, all my HTTP and HTTPs routes to containers is done with docker labels.

Here is John's script adjusted to my Windows Machine:

```powershell
$ports = @(80, 443);

$wslAddress = bash.exe -c "ifconfig eth0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}'"

if ($wslAddress -match '^(\d{1,3}\.){3}\d{1,3}$') {
  Write-Host "WSL IP address: $wslAddress" -ForegroundColor Green
  Write-Host "Ports: $ports" -ForegroundColor Green
}
else {
  Write-Host "Error: Could not find WSL IP address." -ForegroundColor Red
  exit
}

$listenAddress = '0.0.0.0';

foreach ($port in $ports) {
  Invoke-Expression "netsh interface portproxy delete v4tov4 listenport=$port listenaddress=$listenAddress";
  Invoke-Expression "netsh interface portproxy add v4tov4 listenport=$port listenaddress=$listenAddress connectport=$port connectaddress=$wslAddress";
}

$fireWallDisplayName = 'WSL' ;
$portsStr = $ports -join "," ;

Invoke-Expression "Remove-NetFireWallRule -DisplayName $fireWallDisplayName";
Invoke-Expression "New-NetFireWallRule -DisplayName $fireWallDisplayName -Direction Outbound -LocalPort $portsStr -Action Allow -Protocol TCP";
Invoke-Expression "New-NetFireWallRule -DisplayName $fireWallDisplayName -Direction Inbound -LocalPort $portsStr -Action Allow -Protocol TCP";
```

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

