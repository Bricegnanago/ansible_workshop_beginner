# ansible_workshop_beginner

# Processus de configuration

## Acheter un VPS
Acheter un VPS sur Hostinger


## Configuer le serveur
Configuraer le serveur vise à installer les packagess necessaire pour la pratique de Ansible
Voici les actions à effectuer

### Installer les packages
* mise a jour des package
* docker
* openssh-server


### Actions
#### Creer un nouveau container avec l'image de Ubuntu

```bash
docker run -it --name ubuntu_server -d ubuntu:latest
```

#### Installer openssh-server

```bash
# installer openssh-server
apt install openssh-server
```

> Activer les connection ssh dans le container ubuntu. Il s'agit là d'authoriser le server principal à se connecter au container

```bash
# Verifier si le service ssh est activé dans le container
service --status-all
```

```bash
# Definir le mot de passe root du container
passwd root
```

```md
# Ouvrir le fichier /etc/ssh/sshd_config
# Ajouter une nouvelle ligne
PermitRootLogin yes
```

```bash
# Demarrer le service ssh dans le container docker Ubuntu
service ssh start 
```

```bash
# Verifier si le service ssh est activé dans le container
service --status-all
```

```bash
# Ajouter une nouvelle
service ssh start 
```

#### Se connecter au container depuis le VPS

```bash
# Récupérer l'IP du container
hostname -i
```


#### Configurer le container almalinux
##### Installer les packages
```bash
# Verifier s'il y a des mise à jour package
sudo dnf check-update
```

```bash
# Mettre à jour les packages
sudo dnf update -y
```

```bash
# Mettre a jour un package specifique
sudo dnf update <packagename>
```

```bash
sudo dnf install packagename
```
##### Changer le mot de passe du root

Sur AlmaLinux (comme sur d’autres distributions basées sur RHEL/CentOS), si la commande passwd retourne command not found, cela signifie que le paquet contenant cette commande n’est pas installé.

```bash
sudo dnf install -y passwd
```

```bash
dnf --version
```

```bash
# Mettre a jour un package specifique
passwd root
```


##### Generer les clé RSA 

```bash
ssh-keygen -A
```


#### Rendre accessible les serveur slaver depuis 

##### Genérer une clé SSH au niveau du VPS

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

##### Copier ces clés générés dans les container

```bash
# Ubuntu
ssh-copy-id -i /root/.ssh/id_ed25519.pub 172.17.0.2

# Debian
ssh-copy-id -i /root/.ssh/id_ed25519.pub 172.17.0.3

# Almalinux
ssh-copy-id -i /root/.ssh/id_ed25519.pub 172.17.0.4
```


##### Genérer une clé SSH qui sera utilisé par ansible

```bash
ssh-keygen -t ed25519 -C "ansible"
```
> NB: ne pas donner de passphrase pour cette clé ansible. En effet, Ansible devrait pouvoir se connexion automatiquement 
> aux differents serveurs


##### Copier la clé ansible dans les containers
```bash
# Ubuntu
ssh-copy-id -i /root/.ssh/ansible.pub 172.17.0.2

# Debian
ssh-copy-id -i /root/.ssh/ansible.pub 172.17.0.3

# Almalinux
ssh-copy-id -i /root/.ssh/ansible.pub 172.17.0.4
```

##### Se connecter aux serveur via la clé ansible

```bash
# Ubuntu
ssh -i /root/.ssh/ansible 172.17.0.2

# Debian
ssh -i /root/.ssh/ansible 172.17.0.3

# Almalinux
ssh- -i /root/.ssh/ansible.pub 172.17.0.4
```


#### Ajouter un Agent ssh a la master 

```bash
eval $(ssh-agent)
ps aux | grep 26907
ssh-add

alias sshag='eval $(ssh-agent) && ssh-add'
```


#### Creer un repository github


#### Installer Ansible

```bash
sudo apt update
sudo apt install -y software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install -y ansible
ansible --version

# installer plocate
apt install plocate
```

#### Creer un fichier inventory.ini
Le fichier inventory nous permet de faire l'inventaire de nos ressources

Ajouter les lignes suivantes dans le fichier:

```txt
172.17.0.2
172.17.0.3
172.17.0.4
```

#### Tester la connexion au serveur via ansible

Pour tester la connexion aux serveurs via ansible

```bash
ansible all --key-file /root/.ssh/ansible -i inventory.ini -m ping
```


Pour eviter de toujours renseigner le fichier de la clé ssh et le fichier inventory.ini correspondant il est interessant de creer un fichier de configurat
uration ansible.cfg

```bash
touch ansible.cfg
```

Ajouter les lignes suivantes au fichier de configuration

```cfg
[defaults]
inventory = inventory.ini
private_key = ~/.ssh/ansible
```

```bash
ansible all -m ping 
```


#### Effectuer une commande sur plusieurs serveurs à la fois

ansible all -m shell -a 'la commande shell' 

Voici un exemple:

```shell
ansible all -m shell -a "cat /etc/*release"
```

#### Visualiser la liste de tous mes serveurs

```bash
ansible all --list-hosts
```
