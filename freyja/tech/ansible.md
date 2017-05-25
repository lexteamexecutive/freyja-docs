Retour sur la [page d'acceuil](http://docs.lexteam-executive.com/)

---

# Ansible - Sommaire
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

[Ansible](https://www.ansible.com/) est un outil open-source développé en **Python 2.7** permettant
d'automatiser les actions.

La force d'**Ansible** réside en plusieurs
points:
* Il possède un grand nombre de
[modules](https://docs.ansible.com/ansible/modules_by_category.html)
permettant de faire des commandes simple.
* Il supporte un système de
variable utilisant la syntaxe
[Jinja2](http://jinja.pocoo.org/docs/2.9/).
* Il peut déployer sur un
ou plusieurs clients, tournant sur des systèmes différents

**N.B**: Les informations sur cette page concernant **Ansible** ne sont absoluement pas exhaustive.

Prérecquis
----------
Puisqu'**Ansible** est un outil qui doit se connecter vers des hôtes
distant, il existe des prérecquis à la fois pour l'hôte qui déploie (_serveur_), mais aussi sur l'hôte sur lequel le déploiement est effectué (_client_).

### Côté serveur

Afin de pouvoir utiliser **Ansible**, il faut:
* Avoir [installé
Python2.7](https://wiki.python.org/moin/BeginnersGuide/Download)
* Avoir [installé
Ansible](https://docs.ansible.com/ansible/intro**installation.html)

### Côté client

Pour qu'**Ansible** puisse se connecter sur l'hôte et effectuer les actions, il faut au préalable:
* Avoir [installé
Python2.7](https://wiki.python.org/moin/BeginnersGuide/Download)
* Avoir un serveur SSH fonctionnel.
* Que l'utilisateur défini dans le **playbook** puisse se connecter sur le(s) client(s).

---

# Comment ça marche ?

## La théorie

Afin de savoir quelles actions mener et sur quels serveurs, **Ansible** nécessite plusieurs élèments:
* Un **fichier d'inventaire**
* Un **playbook**
* Des **rôles**

### Fichier inventaire

Le **fichier d'inventaire** est, comme son nom l'indique, un fichier contenant le(s) client(s) sur le(s)quel(s) Ansible doit déployer le playbook.

Il peut être défini de manière génèrale dans le dossier ```/etc/ansible/hosts```, ou alors défini séparamment pour chaques **playbooks**.

Le **fichier d'inventaire** peut contenir:
* Des noms d'hôte FQDN (_monserveur.mondomaine.com_)
* Des adresses IP (_192.168.1.20_)
* Des groupes (_[mongroupe]_)

Voici un exemple d'un **fichier d'inventaire**  valide:
```
192.168.1.20
monserveurB.mondomaine.com

[mongroupedeclients]
192.168.1.23
monserveurB.mondomaine.com
monserveurC.mondomaine.com
```

### Playbook

**Playbook**, **playbook**, **playbook**... Depuis tout à l'heure, vous lisez ce terme, sans pour autant en comprendre son sens ...

Un **playbook** est un fichier au format [YML](http://www.yaml.org/), qui permet d'indiquer:
* Le(s) client(s) sur lesquel(s) **Ansible** doit éffectuer les actions.
* L'utilisateur avec lequel **Ansible** doit se connecter sur le(s) client(s).
* Les différents rôles qui serons éxecuter par ce playbook.
* Définir des variable(s) qui pourrons ensuite être utilisées par les roles.

Voici un exemple d'un fichier **playbook** valide (**N.B** Attention à l'identation, le format YML y est sensible ;))
```
hosts: mongroupedeclients
remote**user: monuser-client
roles:
  - role: MAJ
vars:
  DOSSIER-PARTAGE=/Mon/Dossier
```

### Rôle

Un **rôle** est un fichier au format **YML**  qui est appelé par un **playbook**.

Il contiens une liste d'action à effectuer, qui sont généralement composer comme ceci:
* D'un nom de tâche
* Un module Ansible à utiliser

Chaque modules possède des options qui lui sont propre.

Il est possible de rajouter des tests aux tâches afin qu'elles ne s'éxecutent que si le système d'exploitation est d'une famille particulière par exemple.

Aussi, il est possible d'utiliser des variables définies en amont.

Voici un exemple d'un fichier **rôle** valide (**N.B** Attention à l'identation, le format YML y est sensible ;)):
```
---
- name: Mise à jour des logiciels présent
  apt:
    upgrade: yes
    update__cache: yes
  become: yes

- name: Installation d'une liste de logiciels
  apt:
    name: "{{ item }}"
    state: present
  with**items:
    - logiciel1
    - logiciel 2
  become: yes

- name: Copie du dossier partager
  copy:
    src: ./dossier
    dest: "{{ DOSSIER-PARTAGE }}"
```

---

## La pratique

### Comment bien faire son playbook

Pour que le **playbook** puisse s'éxecuter comme il se doit, et qu'il puisse appeller les **roles**, il faut une arborescence bien défini.

Pour appeller un **role**, un **playbook** regarde toujours dans le dossiers ```roles``` puis dans le dossier avec le nom du **role**, puis dans le dossier ```tasks```,pour enfin chercher le fichier ```main.yml```.

Par exemple, si je souhaite que mon **playbook** execute le **role** "MAJ", voici l'arborescence attendue:

```
./    #Dossier de base
./mon-playbook.yml    #Playbook
./roles/    # Dossier qui contiendra tout les roles
./roles/MAJ    # Dossier possédant le même nom que le role
./roles/MAJ/tasks    # Dossier qui contiendra le fichier role
./roles/MAJ/tasks/main.yml    # Fichier role qui contient les actions à executer
```

De plus, il est conseillé de rajouter un dossier ```production``` qui contiendra  le **fichier d'inventaire**.

### Execution

Une fois que vous avez fini votre playbook et que l'arborescence est bonne, vous pouvez vous lancer.

Dans cette exemple, nous partons du principe que nous avons un **playbook** qui se nomme MAJ.yml, et que nous avons un **fichier d'inventaire** qui se trouve dans ```production/hosts```

Placez vous d'abord dans le dossier dans lequel se trouve votre fichier **playbook** , puis exécuter la commande suivante:

```ansible-playbook -i production/hosts MAJ.yml ```

L'option ```-i``` permet d'indiquer le **fichier d'inventaire** que nous souhaitons utiliser.

---

# Les playbooks pour FREYJA

Nous ne traiterons pas dans cette documentation les différents playbooks utilisée par FREYJA.

Pour avoir plus d'informations concernant ces playbooks, rendez vous sur la page [github](https://github.com/lexteamexecutive/freyja-ansible)

---
Retour sur la [page d'acceuil](http://docs.lexteam-executive.com/)
