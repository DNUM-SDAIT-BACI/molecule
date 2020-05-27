# -- Description new structure CI --

## 1. Conception du role

### 1.1. Création du role

> A faire: realiser un role template

- Création d'un rôle

```bash
# initialiser le role
molecule init role my-new-role
# ou créer un role depuis un template
molecule init role my-new-role --template role-template
cd my-new-role
```

- Edition du fichier molecule/default/molecule.yml

```yml
---
dependency:
  name: galaxy
driver:
  name: docker
platforms:
  - name: INSTANCE_NAME # RENAME le nom du conteneur
    image: docker.io/pycontribs/centos:7 # donner l'url de l'image
    command: /sbin/init
    privileged: True # donne tous les droits à docker
    pre_build_image: true
provisioner:
  name: ansible
verifier:
  name: ansible
```

- Editer le role ansible (laisser par défaut dans le cas de la création d'un role template)
  - tasks
  - handlers
  - templates
  - vars
  - etc.

- Réalisation de tests sur le role

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

### 1.2. Validation des tests du role

- Envoyer le role dans une branche de test de gitlab

```bash
# procédure à concevoir
# TaGuEr les roles une fois les tests validés pour qu'ils soient réutilisables

# Autoriser la CI à pouvoir lire la branche de test, et la merge dans le master avec un TAG
git tag -a v1.4 -m "my version 1.4"

# voir https://git-scm.com/book/en/v2/Git-Basics-Tagging

# a revoir, car pas idée claires sur ce sujet.
# l'idée est de ne plus utiliser la DML
```

> need brainstorming

## 2. Création du service

### 2.1. Création du service appellant des roles

- Création d'un service

```bash
# initialiser le role
molecule init role my-new-service
# ou créer un role depuis un template
molecule init role my-new-service --template role-template
cd my-new-service
```

- Edition du fichier molecule/default/molecule.yml

```yml
---
dependency:
  name: galaxy
  options:
    ignore-certs: True
    ignore-errors: True
    role-file: requirements.yml # ici sont indiqués les roles à utiliser
driver:
  name: docker
platforms:
  - name: INSTANCE_NAME # renommer le service
    image: docker.io/pycontribs/centos:7
    command: /sbin/init
    privileged: True # donne tous les droits à docker
    pre_build_image: true
provisioner:
  name: ansible
verifier:
  name: ansible
```

> le fichier requirements va contenir les roles a integrer dans le service

- Editer/créer le fichier requirements.yml (à la racine du rôle)

```bash
# remplir les pre requis

# from GitHub|Gitlab
- src: https://github.com/bennojoy/nginx

# from GitHub|Gitlab, overriding the name and specifying a specific tag
- name: nginx_role
  src: https://github.com/bennojoy/nginx
  version: master

# from a webserver, where the role is packaged in a tar.gz
- name: http-role-gz
  src: https://some.webserver.example.com/files/master.tar.gz
```

- Editer le fichier molecule/default/converge.yml

Y inclure les roles à jouer (ceux du fichier requirements)

```yml
- name: Converge
  hosts: all
  tasks:
    - name: "Include install_squid"
      include_role:
        name: "install_squid"
    - name: "install ntp"
      include_role:
        name: "ansible-role-ntp"
```

- Realiser des tests sur l'installation molecule/default/verify.yml

Exemple:

```yml
- name: Verify
  hosts: all
  tasks:
  - name: Wait for port to come up
    wait_for:
      port: "5260"
      timeout: 60
      host: 0.0.0.0
```

Lancer le verify:

```bash
molecule verify
```

## Autres merdes à regarder

kaniko, packer

tester un putain de tenant!

