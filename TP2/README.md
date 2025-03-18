# Part I : Programmatic approach

# I. Premiers pas

🌞 **Créez une VM depuis le Azure CLI**

```bash
az vm create --resource-group Léo --name az --image Ubuntu2204 --admin-username azureuser --ssh-key-values ~/.ssh/id_rsa.pub --public-ip-sku Standard
```

🌞 **Assurez-vous que vous pouvez vous connecter à la VM en SSH sur son IP publique.**

Récupération IP et connexion ssh :

~~~bash
C:\Users\Utilisateur>az vm show -d -g Léo -n az --query publicIps -o tsv
20.204.148.12

C:\Users\Utilisateur>ssh azureuser@20.204.148.12

azureuser@az:~$
~~~

Vérification des services :

~~~bash
azureuser@az:~$ systemctl status walinuxagent
● walinuxagent.service - Azure Linux Agent
     Loaded: loaded (/lib/systemd/system/walinuxagent.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2025-03-18 08:50:28 UTC; 11min ago
    [...]


azureuser@az:~$ systemctl status cloud-init
● cloud-init.service - Cloud-init: Network Stage
     Loaded: loaded (/lib/systemd/system/cloud-init.service; enabled; vendor preset: enabled)
     Active: active (exited) since Tue 2025-03-18 08:50:28 UTC; 12min ago
     [...]


azureuser@az:~$ cloud-init status
status: done
~~~

# II. Un ptit LAN

🌞 **Créez deux VMs depuis le Azure CLI**

Création d'un réseau virtuel :

~~~bash
C:\Users\Utilisateur>az network vnet create --resource-group Léo --name VNet --address-prefix 10.0.0.0/16 --subnet-name Subnet --subnet-prefix 10.0.0.0/24
~~~

Création des 2 VMs :

~~~bash
C:\Users\Utilisateur>az vm create --resource-group Léo --name az --image Ubuntu2204 --admin-username azureuser --ssh-key-values ~/.ssh/id_rsa.pub --public-ip-sku Standard --vnet-name VNet --subnet Subnet

C:\Users\Utilisateur>az vm create --resource-group Léo --name az2 --image Ubuntu2204 --admin-username azureuser --ssh-key-values ~/.ssh/id_rsa.pub --public-ip-sku Standard --vnet-name VNet --subnet Subnet
~~~

Vérification des IP et ping :

~~~bash
azureuser@az:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 60:45:bd:72:c2:46 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.4/24 metric 100 brd 10.0.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::6245:bdff:fe72:c246/64 scope link
       valid_lft forever preferred_lft forever


azureuser@az2:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 7c:ed:8d:27:22:f3 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.5/24 metric 100 brd 10.0.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::7eed:8dff:fe27:22f3/64 scope link
       valid_lft forever preferred_lft forever


azureuser@az:~$ ping 10.0.0.5
PING 10.0.0.5 (10.0.0.5) 56(84) bytes of data.
64 bytes from 10.0.0.5: icmp_seq=1 ttl=64 time=1.15 ms
64 bytes from 10.0.0.5: icmp_seq=2 ttl=64 time=0.823 ms
64 bytes from 10.0.0.5: icmp_seq=3 ttl=64 time=0.653 ms
64 bytes from 10.0.0.5: icmp_seq=4 ttl=64 time=0.870 ms
^C
--- 10.0.0.5 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3076ms
rtt min/avg/max/mdev = 0.653/0.875/1.154/0.180 ms
~~~
