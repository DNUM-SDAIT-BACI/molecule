# Installation d'un environnement de CI

> Cet environnement permet d'effectuer des tests de roles ansible dans divers environnements: centos/ubuntu via vagrant/docker

## Preconf de la Centos 7/8

Mettre en place un compte avec lequel les tests seront déroulés

- Créer l'utilisateur avec lequel les tests seront effectués (en root)

```bash
# ajouter l'user deploy ou autre, peu importe, débrouille toi!
adduser deploy
```

- Editer le sudoers /etc/sudoers (en root)

```bash
# See sudoers(5) for more information on "#include" directives:
deploy  ALL=(ALL:ALL) NOPASSWD:ALL
```

- Ajouter les paquets suivants

```bash
sudo yum install vim
```

- Disable le firewall (notamment pour la centos8) pour que docker fonctionne

```bash
sudo systemctl disable firewalld
```



## CENTOS7 amuses toi bien omg!

Si ansible est installé par défaut, mais ceci est inutile sur la CI

```bash
# décommenter les lignes suivantes du fichier /etc/ansible/ansible.cfg
host_key_checking = False
retry_files_enabled = False
```

## Installation de docker

- Installation sous CENTOS 7

```bash
# installer et start docker
sudo yum install docker
sudo systemctl enable docker
sudo systemctl start docker
```

- Installation sous CENTOS 8

```bash
# Ajout du repo docker-ce
sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
# checker si le paquet est disponible
sudo dnf list docker-ce
# installer la dernière version de docker
sudo dnf install docker-ce --nobest -y
# Concernant un problème de dépendances des centos8, installer le paquet container
sudo dnf install https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
# enable/start docker
sudo systemctl enable docker
sudo systemctl start docker
```

- Configuration pour CENTOS 7

```bash
# Editer/Créer le fichier /etc/docker/daemon.json
# autoriser les user dans le group dockerroot 
{
    "live-restore": true,
    "group": "dockerroot"
}

# Ajout du user deploy dans le group dockerroot
sudo usermod -aG dockerroot deploy

# Redemarrer docker
systemctl restart docker
```

- Configuration pour CENTOS 8

```bash
# Editer/Créer le fichier /etc/docker/daemon.json
# autoriser les user dans le group docker
{
    "live-restore": true,
    "group": "docker"
}

# Ajout du user deploy dans le group docker
sudo usermod -aG docker deploy

# Redemarrer docker
systemctl restart docker
```

## Installation de vagrant & virtualbox sur centos7 (servira a molecule): NE PAS FAIRE

```bash
# installation de vbox https://linuxize.com/post/how-to-install-virtualbox-on-centos-7/
sudo yum install kernel-devel kernel-headers make patch gcc wget
# ajouter le repo de virtual box, ou les stocker dans pulp/dml... js'sais pas à voir
sudo wget https://download.virtualbox.org/virtualbox/rpm/el/virtualbox.repo -P /etc/yum.repos.d
yum install VirtualBox-6.1.x86_64

# installer vagrant, pareil prévoir un repo ou un rep dans la DML
sudo yum install https://releases.hashicorp.com/vagrant/2.2.7/vagrant_2.2.7_x86_64.rpm

# creer l'env de vagrant pour tester le dev
mkdir ~/my-vagrant-project
cd ~/my-vagrant-project
# pareil, toper une vbox à caler dans la DML, configurer la source y tout, bref oseb
# voir https://www.vagrantup.com/intro/getting-started/boxes.html pour les url de download
vagrant init centos/7
vagrant box add centos/7
vagrant up # download l'image de vagrant vbox, qu'il faudra mettre dans la DML au besoin

# ajout de la box et demarrage de celle ci
vagrant box add centos7 centos7.box
mkdir ma_vm && cd ma_vm
vagrant init centos7
vagrant up
# Once you have downloaded the file, you need to add the box to your vagrant config:
vagrant box add --name <name of the box> --box-version <version of the box> <downloaded box file>

```


## Installation de molecule

- Installation sous `CENTOS 7`

```bash
# bonne méthode!!!
sudo yum install epel-release
sudo yum update -y && sudo reboot
sudo yum install python-devel python-setuptools python-pip
sudo pip install --upgrade pip
sudo pip install virtualenv
# revenir avec le compte user dans lequel vous voulez faire du virtual env
cd ~
virtualenv djangoenv
source ~/djangoenv/bin/activate
# installer molecule dans le virtual env
pip install molecule
pip install docker # pour avoir le plugin docker
pip install python-vagrant # pour avoir le plugin vagrant
molecule --version
```

- Installation sous `CENTOS 8`

```bash
# installation des paquets de base
sudo dnf update && sudo reboot
sudo dnf install gcc python3-pip python3-devel openssl-devel libffi-devel git
# Creation d'un venv sous le compte user deploy
# creation du home du venv
cd ~
python3 -m venv python3-virtualenv
# Activation du venv
source python3-virtualenv/bin/activate
# Maj de pip dans le venv
pip install --upgrade pip
# installation de molecule dans le venv
pip install molecule
pip install docker
```





## Test de molecule


```bash
# creer un role
molecule init role my-new-role
# creer un scenario du nom de tutu
cd my-new-role
# verifier le fonctionnement de docker:
docker run hello-world  
# Now, we can tell Molecule to create an instance with:
molecule create
# We can verify that Molecule has created the instance and they’re up and running with:
molecule list
# Now, let’s add a task to our tasks/main.yml like so:
- name: Molecule Hello World!
  debug:
    msg: Hello, World!
# We can then tell Molecule to test our role against our instance with:
molecule converge
# If we want to manually inspect the instance afterwards, we can run:
molecule login
# We now have a free hand to experiment with the instance state.
# Finally, we can exit the instance and destroy it with:
molecule destroy

# full test sequence:
molecule test
```

## Paramétrage avancés molecule

- commandes avancées

```bash
# voir https://molecule.readthedocs.io/en/latest/usage.html#dependency

# Initialiser un role depuis un template molecule, cela peut servir à avoir des templates a usage différent (docker ou vagrant par ex)
molecule init role foo --template path

# creer un scenario avec vagrant (il servent a tester un role dans différents environnements):
cd squid_vagrant
molecule init scenario -r install_squid --driver-name vagrant squid_vagrant
```

## Configurer un role avec vagrant from scratch (useless)

> prerequis: avoir un environnement vagrant fonctionnel

> pour les images rhel8, https://access.redhat.com/articles/4238681

- Creer le nouveau role via molecule

```bash
# pour les exemples https://github.com/ansible-community/molecule-vagrant/tree/master/molecule_vagrant
# creer un role avec comme sc par défaut vagrant
molecule init role my-new-role --driver-name vagrant
# entrer dans le rep du role
cd my-new-role
```

- Editer molecule/default/molecule.yml

```yml
---
dependency:
  name: galaxy
driver:
  name: vagrant
  provider:
    name: virtualbox
platforms:
  - name: NOM_DU_MODULE
    box: NOM_DE_LA_BOX # vagrant box list
    memory: 1024 # memoire
    cpus: 1 # nb cpu
provisioner:
  name: ansible
verifier:
  name: ansible
```

- Editer molecule/default/converge.yml pour y ajouter le droit d'execution en tant que root

```yml
---
- name: Converge
  hosts: all
  become: true # AJOUTER CETTE LIGNE
  tasks:
    - name: "Include my-new-role"
      include_role:
        name: "my-new-role"
```

- Configurer le role

A vous de voir... ajouter des tasks tout ça tout ça

- Effectuer le test du role 

```bash
molecule test
```

## Configurer un role avec docker from scratch

- Executer un role dans un conteneur docker et utiliser systemd, editer le fichier molecule/default/molecule.yml

```yml
---
dependency:
  name: galaxy
driver:
  name: docker
platforms:
  - name: instance
    image: docker.io/pycontribs/centos:7
    #image: docker.io/pycontribs/centos:8 # fonctionnel aussi
    command: /sbin/init
    privileged: True # donne tous les droits à docker
    pre_build_image: true
provisioner:
  name: ansible
verifier:
  name: ansible # celui à utiliser par defaut, meme s'il existe testinfra
  ```

- Configurations specifique docker

https://molecule.readthedocs.io/en/latest/configuration.html#docker

- Jouer un scenario dans un role

```bash
molecule create|test|... --scenario-name foo
```

- Ajouter des requirements à un role

```yml
# from galaxy
- name: yatesr.timezone

# from GitHub
- src: https://github.com/bennojoy/nginx

# from GitHub, overriding the name and specifying a specific tag
- name: nginx_role
  src: https://github.com/bennojoy/nginx
  version: master

# from a webserver, where the role is packaged in a tar.gz
- name: http-role-gz
  src: https://some.webserver.example.com/files/master.tar.gz

# from a webserver, where the role is packaged in a tar.bz2
- name: http-role-bz2
  src: https://some.webserver.example.com/files/master.tar.bz2

# from a webserver, where the role is packaged in a tar.xz (Python 3.x only)
- name: http-role-xz
  src: https://some.webserver.example.com/files/master.tar.xz

# from GitLab or other git-based scm, using git+ssh
- src: git@gitlab.company.com:mygroup/ansible-base.git
  scm: git
```


