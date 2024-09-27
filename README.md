# Método 01 - Raspberry Pi

- Instalar Docker Engine 
- Executar o container ./diretorio do docker-compose.yml `docker compose up -d`

![](https://github.com/esfonseca/nginx-rtmp-docker-compose/blob/master/img/nginx-rtmp_OBS-RASPBERRYMODE.drawio.png)

# Método 02 - WSL2 Ubuntu

- Realizar Liberações no Firewall
- [Instalar o WSL no Windows](./wsl2-devops-configuration.md)
- Fazer o PortProxy no Host para o WSL
- Executar o container no WSL ./diretorio do docker-compose.yml `docker compose up -d`

![](https://github.com/esfonseca/nginx-rtmp-docker-compose/blob/master/img/nginx-rtmp_OBS-WSL-MODE.drawio.png)

## Liberar Firewall Windows

- Liberar porta TCP 1935 de Entrada

        New-NetFirewallRule -DisplayName "Allow TCP 1935" -Direction Inbound -Protocol TCP -LocalPort 1935 -Action Allow -RemoteAddress 0.0.0.0/0

- Verifica Escuta da porta `netstat -an | findstr 1935`

          TCP    0.0.0.0:1935           0.0.0.0:0              LISTENING
          TCP    [::1]:1935             [::]:0                 LISTENING

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

- Comando `netsh interface portproxy show v4tov4` lista endereços de entrada 0.0.0.0 (Todos) e endereço de destino `IP do WSL`

        Escuta em ipv4:             Conectar-se a ipv4:

        Endereço        Porta       Endereço        Porta
        --------------- ----------  --------------- ----------
        0.0.0.0         1935        172.28.54.51    1935

### OBS Windows de Transmissão (localhost)

- Importar `CENA_RTMP_EXEMPLO.json` para dentro do OBS

![](https://github.com/esfonseca/nginx-rtmp-docker-compose/blob/master/img/import_scene_obs.png)

- Cena importada no OBS

![](https://github.com/esfonseca/nginx-rtmp-docker-compose/blob/master/img/cena_exemplo_obs.gif)

## Configuração de Transmissão RTMP - DJI Fly

- Endereço RTMP colocar o IP do Docker Host

![](https://github.com/esfonseca/nginx-rtmp-docker-compose/blob/master/img/app_dji_fly.jpg)

## Observações de desempenho (Método 02 - WSL2 Ubuntu)

Consumo do container com `docker stats` de 61Mb de memória e média de 13% de CPU (Core i7 13700H) para uma trasmissão 3Mbps FULL HD.

- 1 OBS no Host Windows consumindo o streaming
- 1 Smarphone com VLC consumindo o streamig

## Equipamento utilizados

| Equipamentos             | Windows - WSL2 | Raspberry Pi |
|--------------------------|----------------|--------------|
| Drone DJI Air2S          | X              | X            |
| Controle DJI RC2         | X              | X            |
| Router Gigabit 5GHz      | X              | X            |
| Raspberry Pi 5           |                | X            |
| W11 Host PC (Core i7 13700H) | X              |              |

## Softwares 

- W11 23H2 22631.4169
- OBS Studio : https://obsproject.com/pt-br/download
- Ciclano Multi Streaming Plataform (Paid) - https://app.ciclano.io/