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

# Part II : cloud-init

## 2. Gooooo

🌞 **Tester `cloud-init`**

Création de la VM avec cloud-init qui créer un user cmeyer :

~~~bash
az vm create --resource-group Léo --name cloudinit-vm --image Ubuntu2204 --admin-username azureuser --ssh-key-values ~/.ssh/id_rsa.pub --custom-data C:\Users\Utilisateur\OneDrive\Bureau\TP_Leo\cloud_init.txt --public-ip-sku Standard
~~~

🌞 **Vérifier que `cloud-init` a bien fonctionné**

~~~bash
C:\Users\Utilisateur>ssh cmeyer@20.244.125.121
The authenticity of host '20.244.125.121 (20.244.125.121)' can't be established.
[...]

cmeyer@cloudinit-vm:~$
~~~

## 3. Write your own

🌞 **Utilisez `cloud-init` pour préconfigurer la VM :**

cloud-config.txt :

~~~bash
#cloud-config
users:
  - name: cup82idon
    sudo: ALL=(ALL) NOPASSWD:ALL
    shell: /bin/bash
    groups: sudo, docker
    ssh_authorized_keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDMv69bK3zGwHlrUsdyY8RGJas1nufp0Kypn7Dp87W7geLmJsxjvyiwyE4T9Ed1ylD6jPWJFUdBl1ppBSf5UH/KrJHvkCr73o3u3O+giuHTCeRX3RVm5pdbMRFNObM9z+fiwWLE8T7p4l1ukAvKskELgvPImcFqdGI2QxgWLFPQNGF1FqDFj66OtNq0ulqTEIPDU6xOSFnxhsYinqYm5ShXHROWyEUqqSaEqyutXk3aK0pitX43e1YKO9YhkCsH18FEJQCIYmdjIZivcpfMOdeILhRG5NtvGceBiO6lNxR6htbjN6C4SzxIs2dkBQKoJ2t3Cn4+aGKyYUEBC9vGAC2P/sGBQGFUkMaGAHpX7wS7qUV8ydCLxCqUNoQWOzbvfaz54TybjTxeq3fAmN9w9VPQGr4E7hCGlCwWEEsmA5+0aL/aAVb/uHmEokEQ5OtoqhzzeJ9+NV6JbtItfuokiwFDdC9b+KKe6sAt/iFffPMvVK8Y79aN3HCLq2ucgbNFaPq6hlQPYLEcxgE/qF7w+PNfxL/BVL7M/YXPHH+Oa+H4cbHIDCUyHrE3EtZE/HaDYtOH5t73YcS+wn5TW1VqavAu7GlQOmKjPV8OXKeL8yikoaFcJ80ecG8nD6PfsCqjuty8LHAa3SULwMlbazT8RLBLc983FBgCxu/TYvdCRXN12w== utilisateur@Pc-Cup
    passwd: $6$hZS0BI.6qGRuSilj$FaAQJpQNK9NxVT9KT0EfLB8SlAJWRYCORV0D5AGhXyOTciw/ZZWFf1mPDS5ZzNLpfPIOI9Bvki/Zwhna95Hmn.

package_update: true
package_upgrade: true

runcmd:
  - apt-get update && apt-get install -y docker.io
  - usermod -aG docker cup82idon
  - systemctl enable --now docker
  - docker pull alpine:latest
~~~

Création de la VM et connexion avec le user cup82idon :

~~~bash
C:\Users\Utilisateur>az vm create --resource-group Léo --name cloudinit-docker --image Ubuntu2204 --admin-username azureuser --ssh-key-values ~/.ssh/id_rsa.pub --custom-data C:\Users\Utilisateur\OneDrive\Bureau\TP_Leo\cloud-init.txt --public-ip-sku Standard


C:\Users\Utilisateur>ssh cup82idon@74.225.203.211
The authenticity of host '74.225.203.211 (74.225.203.211)' can't be established.
[...]
cup82idon@cloudinit-docker:~$
~~~

# Part III : Terraform

🌞 **Constater le déploiement**

~~~bash
PS C:\Users\Utilisateur\OneDrive\Bureau\TP_Leo\Terraform> az vm list -o table
Name    ResourceGroup    Location      Zones
------  ---------------  ------------  -------
tp2-vm  TP2-RESOURCES    westeurope


PS C:\Users\Utilisateur\OneDrive\Bureau\TP_Leo\Terraform> az vm show --name tp2-vm --resource-group TP2-RESOURCES -o table
Name    ResourceGroup    Location    Zones
------  ---------------  ----------  -------
tp2-vm  TP2-RESOURCES    westeurope


PS C:\Users\Utilisateur\OneDrive\Bureau\TP_Leo\Terraform> az group list -o table
Name              Location      Status
----------------  ------------  ---------
tp2-resources     westeurope    Succeeded
~~~

➜ **Autres commandes Terraform**

```bash
# Vérifier que votre fichier .tf est valide
$ terraform validate

# Formate un fichier .tf au format standard
$ terraform fmt

# Afficher les ressources du déploiement
$ terraform state list

# Afficher les détails d'une des ressources du déploiement
$ terraform state show <RESSOURCE>

# Détruit les ressources déployées
$ terraform destroy
```

## 3. Do it yourself

🌞 **Créer un *plan Terraform* avec les contraintes suivantes**

`main.tf` :

~~~bash
provider "azurerm" {
  features {}
  subscription_id = "f92007d8-4bb1-42ac-8372-f4fe3cfc0ad3"
}

resource "azurerm_resource_group" "main" {
  name     = "${var.prefix}-resources"
  location = var.location
}

resource "azurerm_virtual_network" "main" {
  name                = "${var.prefix}-network"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

resource "azurerm_subnet" "internal" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.2.0/24"]
}

resource "azurerm_public_ip" "node1_pip" {
  name                = "${var.prefix}-node1-pip"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  allocation_method   = "Static"
}

resource "azurerm_network_interface" "node1_nic" {
  name                = "${var.prefix}-node1-nic"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location

  ip_configuration {
    name                          = "public"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.node1_pip.id
  }
}

resource "azurerm_network_interface" "node2_nic" {
  name                = "${var.prefix}-node2-nic"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location

  ip_configuration {
    name                          = "private"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
  }
}

resource "azurerm_network_security_group" "ssh" {
  name                = "${var.prefix}-ssh-nsg"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  security_rule {
    name                       = "allow_ssh"
    priority                   = 100
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    source_address_prefix      = "*"
    destination_port_range     = "22"
    destination_address_prefix = "*"
  }
}

resource "azurerm_network_interface_security_group_association" "node1_assoc" {
  network_interface_id      = azurerm_network_interface.node1_nic.id
  network_security_group_id = azurerm_network_security_group.ssh.id
}

resource "azurerm_linux_virtual_machine" "node1" {
  name                  = "${var.prefix}-node1"
  resource_group_name   = azurerm_resource_group.main.name
  location              = azurerm_resource_group.main.location
  size                  = "Standard_F2"
  admin_username        = "azureuser"
  network_interface_ids = [azurerm_network_interface.node1_nic.id]

  admin_ssh_key {
    username   = "azureuser"
    public_key = file("~/.ssh/id_rsa.pub")
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }

  os_disk {
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }
}

resource "azurerm_linux_virtual_machine" "node2" {
  name                  = "${var.prefix}-node2"
  resource_group_name   = azurerm_resource_group.main.name
  location              = azurerm_resource_group.main.location
  size                  = "Standard_F2"
  admin_username        = "azureuser"
  network_interface_ids = [azurerm_network_interface.node2_nic.id]

  admin_ssh_key {
    username   = "azureuser"
    public_key = file("~/.ssh/id_rsa.pub")
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }

  os_disk {
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }
}
~~~

`variables.tf` :

~~~bash
variable "prefix" {
  description = "Préfixe des ressources Azure"
  default     = "tp2Etape2"
}

variable "location" {
  description = "Région Azure"
  default     = "West Europe"
}
~~~

Connexion en ssh à node1 :

~~~bash
C:\Users\Utilisateur>ssh azureuser@20.4.68.12
The authenticity of host '20.4.68.12 (20.4.68.12)' can't be established.
[...]

azureuser@tp2Etape2-node1:~$
~~~

Connexion en ssh à node2 via node1 :

~~~bash
C:\Users\Utilisateur>ssh -J azureuser@20.4.68.12 azureuser@10.0.2.5
The authenticity of host '10.0.2.5 (<no hostip for proxy command>)' can't 
[...]

azureuser@tp2Etape2-node2:~$
~~~

## 4. cloud-iniiiiiiiiiiiiit

### A. Un premier tf + cloud-init

🌞 **Intégrer la gestion de `cloud-init`**
