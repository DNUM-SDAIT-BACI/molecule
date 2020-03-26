# Installation d'un environnement de CI

> Cet environnement permet d'effectuer des tests de roles ansible dans divers environnements: centos/ubuntu via vagrant/docker

## CENTOS7 amuses toi bien omg!

```bash
# décommenter les lignes suivantes du fichier /etc/ansible/ansible.cfg
host_key_checking = False
retry_files_enabled = False
```

## Installation de gitlab

https://www.howtoforge.com/tutorial/how-to-install-and-configure-gitlab-on-ubuntu-1804/


## Installation de docker

```bash
# installer et start docker
sudo yum install docker
systemctl enable docker
systemctl start docker

# autoriser les user dans le group dockerroot
vim /etc/docker/daemon.json
{
    "live-restore": true,
    "group": "dockerroot"
}

# ajout du user deploy dans le group dockerroot
sudo usermod -aG dockerroot deploy

# redemarrer docker
systemctl restart docker
```

## Installation de vagrant & virtualbox sur centos7 (servira a molecule)

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
molecule init scenario -r squid_vagrant --driver-name vagrant toto
```

## Configurer un role avec vagrant from scratch

> prerequis: avoir un environnement vagrant fonctionnel

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