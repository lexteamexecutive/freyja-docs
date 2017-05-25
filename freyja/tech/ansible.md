# Ansible

# Sommaire
-   [Sommaire](#sommaire)
-   [Introduction](#introduction)
    -   [Prérecquis](#prérecquis)
        -   [Côté serveur](#côté-serveur)
        -   [Côté client](#côté-client)
-   [Comment ça marche ?](#comment-ça-marche-?)
    -   [La théorie](#la-théorie)
        -   [Fichier inventaire](#fichier-inventaire)
        -   [Playbook](#playbook)
        -   [Rôle](#Rôle)
    -   [La pratique](#la-pratique)
        -   [Comment bien faire son playbook](#comment-bien-faire-son-playbook)
        -   [Execution](#execution)
-   [Les playbooks pour FREYJA](#les-playbooks-pour-freyja)
---
# Introduction

[Ansible](https://www.ansible.com/) est un outil open-source développé en _Python 2.7_ permettant
d'automatiser les actions.

La force d'_Ansible_ réside en plusieurs
points:
* Il possède un grand nombre de
[modules]((https://docs.ansible.com/ansible/modules\_by\_category.html)
permettant de faire des commandes simple.
* Il supporte un système de
variable utilisant la syntaxe
[Jinja2](http://jinja.pocoo.org/docs/2.9/).
* Il peut déployer sur un
ou plusieurs clients, tournant sur des systèmes différents

**N.B**: Les informations sur cette page concernant _Ansible_ ne sont absoluement pas exhaustive,.
**N.B 2**: Les mots en **gras** sont des liens vers les sites de documentations.

Prérecquis
----------
Puisqu'_Ansible_ est un outil qui doit se connecter vers des hôtes
distant, il existe des prérecquis à la fois pour l'hôte qui déploie (serveur), mais aussi sur l'hôte sur lequel le déploiement est effectué (client).

### Côté serveur

Afin de pouvoir utiliser _Ansible_, il faut:
* Avoir [installé
Python2.7](https://wiki.python.org/moin/BeginnersGuide/Download)
* Avoir [installé
Ansible](https://docs.ansible.com/ansible/intro_installation.html)

### Côté client

Afin qu'_Ansible_ puisse se connecter sur l'hôte pour effectuer les actions, il faut:
* Avoir [installé
Python2.7](https://wiki.python.org/moin/BeginnersGuide/Download)
* Avoir un serveur SSH fonctionnel.
* Que l'utilisateur défini dans le playbook d'Ansible puisse se connecter sur le(s) client(s).

---

# Comment ça marche ?

## La théorie

Afin de savoir quelles actions mener, et sur quels serveurs, _Ansible_ nécessite plusieurs élèments:
* Un _fichier d'inventaire_
* Un _playbook_
* Des _rôles_

### Fichier inventaire

Le _fichier d'inventaire_ est, comme son nom l'indique, un fichier contenant le(s) client(s) sur lesquels Ansible doit déployer le playbook.

Il peut être défini de manière génèrale dans le dossier ```/etc/ansible/hosts```, ou alors défini séparamment pour chaques _playbooks_, ou a plusieurs endroits en même temps.

Le _fichier d'inventaire_ peut contenir:
* Des noms d'hôte FQDN (monserveur.mondomaine.com)
* Des adresses IP (192.168.1.20)
* Des groupes

Voici un exemple d'un _fichier d'inventaire_  valide:
```
192.168.1.20
monserveurB.mondomaine.com

[mongroupedeclients]
192.168.1.23
monserveurB.mondomaine.com
monserveurC.mondomaine.com
```

### Playbook

_Playbook_, _playbook_, _playbook_... Depuis tout à l'heure, vous lisez ce terme, sans pour autant en comprendre son sens ...

Un _playbook_ est un fichier au format [YML](http://www.yaml.org/), qui permet d'indiquer:
* Le(s) client(s) sur lesquel(s) _Ansible_ doit éffectuer les actions.
* L'utilisateur avec lequel _Ansible_ doit se connecter sur le(s) client(s).
* Les différents rôles qui serons éxecuter par ce playbook.
* Définir des variable(s) qui pourrons ensuite être utilisées par les roles.

Voici un exemple d'un fichier _playbook_ valide (**N.B** Attention à l'identation, le format YML y est sensible ;))
```
hosts: mongroupedeclients
remote_user: monuser-client
roles:
  - role: MAJ
vars:
  DOSSIER_PARTAGE=/Mon/Dossier
```

### Rôle

Un _rôle_ est un fichier au format _YML_  qui est appelé par un _playbook_.

Il contiens une liste d'action à effectuer, qui sont généralement composer comme ceci:
* D'un nom de tâche
* Un module Ansible à utiliser

Chaque modules possède des options qui lui sont propre.

Il est possible de rajouter des tests aux tâches afin qu'elle ne s'éxecutent que si le système d'exploitation est d'une famille particulière par exemple.

Aussi, il est possible d'utiliser des variables défini en amont.

Voici un exemple d'un fichier _rôle_ valide (**N.B** Attention à l'identation, le format YML y est sensible ;)):
```
---
- name: Mise à jour des logiciels présent
  apt:
    upgrade: yes
    update_cache: yes
  become: yes

- name: Installation d'une liste de logiciels
  apt:
    name: "{{ item }}"
    state: present
  with_items:
    - logiciel1
    - logiciel 2
  become: yes

- name: Copie du dossier partager
  copy:
    src: ./dossier
    dest: "{{ DOSSIER_PARTAGE }}"
```

## La pratique

### Comment bien faire son playbook

Pour que le _playbook_ puisse s'éxecuter comme il se doit, et qu'il puisse appeller les _roles_, il faut une arborescence bien défini.

Pour appeler un _role_, un _playbook_ regarde toujours dans le dossiers ```roles``` puis dans le dossier avec le nom du _role_, puis dans le dossier ```tasks```,pour enfin chercher le fichier ```main.yml```.

Par exemle, si je souhaite que mon _playbook_ execute le _role_ "MAJ", voici l'arborescence attendue:

```
./                                                    #Dossier de base
./mon-playbook.yml                   #Playbook
./roles/                                         # Dossier qui contiendra tout les roles
./roles/MAJ                                 # Dossier possédant le même nom que le role
./roles/MAJ/tasks                      # Dossier qui contiendra le fichier role
./roles/MAJ/tasks/main.yml    # Fichier role qui contient les actions à executer
```

De plus, il est conseillé de rajouter un dossier ```production``` qui contiendra  le _fichier d'inventaire_.

### Execution

Une fois que vous avez fini votre playbook et que l'arborescence est bonne, vous pouvez vous lancer.

Dans cette exemple, nous partons du principe que nous avons un _playbook_ qui se nomme MAJ.yml, et que nous avons un _fichier d'inventaire_ qui se trouve dans ```production/hosts```

Placez vous d'abord dans le dossier dans lequel se trouve votre fichier _playbook_ , puis exécuter la commande suivante:
```ansible-playbook -i production/hosts MAJ.yml ```

L'option ```-i``` permet d'indiquer le _fichier d'inventaire_ que nous souhaitons utiliser.

# Les playbooks pour FREYJA

Nous ne traiterons pas dans cette documentation les différents playbooks utilisée par FREYJA.

Pour avoir plus d'informations concernant ces playbooks, rendez vous sur la page github: https://github.com/lexteamexecutive/freyja-ansible
