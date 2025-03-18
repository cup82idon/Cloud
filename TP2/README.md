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


- assurez-vous qu'elles ont une IP privée (avec `ip a`)
- elles peuvent se `ping` en utilisant cette IP privée
- deux VMs dans un LAN quoi !

> *N'hésitez pas à vous rendre sur la WebUI de Azure pour voir vos VMs créées.*

---
