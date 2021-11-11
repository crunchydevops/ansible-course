# Les Playbooks

## Les facts dans un fichier YAML
### Utilisation de when 
```ansible-playbook -i inventory_children ansible_facts_using_when.yml```
### Utilisation de la commande assert 
```ansible-playbook -i inventory_children ansible_facts_using_assert.yml```  

### Le prompt et les conditions
```ansible-playbook -i inventory_children conditions.yml --limit ubuntu```

### les boucles
```ansible-playbook -i inventory_children loops.yml --limit ubuntudocker```

## Passage d'information entre les hosts
### In-memory Inventory 
Pour passer des variables entre remote-to-remote host il est possible
de creer un host de type dummy et lui attacher des variables pour les passer 
vers l'autre host.

```ansible-playbook -i inventory_children runtime_inventory_additions.yml```

### Deploiement d'une cle ssh vers des slave hosts

```shell script
 ansible-playbook -i inventory_children propagate_ssh_key.yml 
```

### Inventaire dynamique
Voir le fichier DYNAMIC_INVENTORY.md


### Utilisation des variables et des filtres
#### Change the Message Of The Day (MOTD) 
cd ..
```ansible-playbook -i inventory_children motd.yml```

#### Vreer son propre filtre 
```ansible-playbook -i inventory_children new_filter.yml --limit target2 ```

#### Exemple de filtre: obtenir la derniere version de glusterfs
```shell script
    ansible-playbook -i inventory_children git_version_filter.yml
```
Dans portainer verifiez la version de glusterfs, sous /home/centos

## Utilisation de Jinja2 avec une installation de glusterfs
Modifiez le fichier inventory_gluster  
faire des ssh-copy-id vers le leader et les slaves   
Check avec une commande ad-hoc  
Lancez le script ```ansible-playbook -i inventory_gluster install_gluster_centos.yml```  
check on leader ```sudo /usr/local/sbin/gluster volume info```
et ```lsblk -f```

### Test sur la version des OS avec une installation de git sur l'ensemble des hosts
```shell script
    ansible-playbook -i inventory_children install_on_multios.yml
```
### Les modules
#### Creer son propre module 
Allez dans votre compte github pour creer un token   
Selectionnez Settings de votre compte github   
et selectionnez **Developer Settings** et ensuite **Personnal Access Tokens**   
Creer un token et lui donner les droits pour creer un repo github.    
Dans votre home directory faire un ```vi token``` et copier votre
token.  
Toujours sous le prompt venv dans votre directory ansible-course
faire ```pip3 install requests``` et 
```ansible-playbook -i inventory_children ansible_create_module.yml```

### Les Roles
#### Mettre le precedement playbook dans un role 
Dans votre home directory sur votre ansible-controller toujours sous le prompt venv
faire ```mkdir example-role```  
et ```cd example-role```  
```ansible-galaxy init github.role```  
creer un ficher playbook.yml    
```yaml
---
- name: use a dedicated Ansible module
  hosts: localhost
  roles:
    - { role: github.role }
```
Dans example-role/github.role/tasks/main.yml 
copier le code suivant
```yaml
# tasks file for github.role
- name: Create a github Repo
  github_repo:
    github_auth_key: "{{ git_key }}"
    name: "repo-create-with-ansible"
    description: "Ansible module for github"
    private: no
    has_issues: no
    has_wiki: no
    has_downloads: no
    state: present
  register: result
```
Dans  example-role/github.role/defaults/main.yml
```yaml
# default file for github.role
git_key: d6f90b4be8axxxxxxxxxxxxxxx
```
Dans votre directory example-role, faire un 
```shell script
   cp -r ../ansible-examples/library . 
   cp -r ../ansible-examples/inventory_children . 
```
Tapez la commande suivante: 
```ansible-playbook -i inventory_children playbook.yml```

### Ansible Vault
Nous allons voir comment crypter nos informations sensibles avec Ansible
Cryptez la variable token dans votre projet example github.role/defaults/main.yml  
Tapez  
```ansible-vault encrypt  main.yml```   
entrez votre mot de passe   
mettrez ce mot de passe dans un fichier  
```vi /home/<home_directory>/mysecret```   
Vous pouvez executer le playbook avec une variable dans le example-role directory   
```ansible-playbook -i inventory_children --vault-password-file ~/mysecret playbook.yml```   
vous pouvez metter le path de ce fichier dans votre ```.bash_profile``` file.  
```export  ANSIBLE_VAULT_PASSWORD_FILE=/home/<home>/mysecret```      
et vous entrez la commande sans vous soucier du fichier du mot de passe  
```ansible-playbook -i inventory_children playbook.yml``` 

### Les roles et comment obtenir du code modulaire
Creez une directory ansible-postgresql sur la machine ansible-controlleur dans la home directory  
Copiez le fichier inventory_gluster dans cette nouvelle directory
Faire un 
```ansible-galaxy init postgresql.role``` 
Creez un fichier playbook.yaml 
```yaml
---
- name: use a dedicated Ansible postgresql role
  hosts: leader
  become: yes
  roles:
    - { role: postgresql.role }
```
Faire la commande Ad-Hoc pour obtenir la distribution et la version de Centos 
```shell
ansible leader -m setup -a "filter=ansible_distribution,ansible_distribution_version "  -i inventory_gluster
```
Dans la directory tasks du role creez le fichier variables.yaml
```yaml
---
# Variables configuration
- name: Include OS-specific variables (Centos)
  include_vars: "{{ ansible_distribution }}-{{ ansible_distribution_version.split('.')[0] }}.yml"
  when: ansible_distribution == "CentOS"

# Mettre ensuite les differentes versions d'OS et versions
```
Ajouter ces lignes dans le fichier main.yml de task
```yaml
---
# tasks file for postgresql.role
- include_tasks: variables.yml

# Setup /install task
- include_tasks: setup-CentOS.yml
  when: ansible_distribution == 'CentOS'

- include_tasks: initialize.yml

- name: Ensure Postgresql is started and enable on boot
  service:
  name: "{{ postgresql_daemon }}"
  state: "{{ postgresql_service_state }}"
  enabled: "{{ postgresql_service_enabled }}"

- import_tasks: users.yml
```

Creez setup-Centos.yml 
```yaml
---
- name: Check if the postgresql packages are installed
  yum:
  name: "{{ postgresql_packages }}"
  state: present

- name: Check if the postgresql librairies are installed
  yum:
  name: "{{ postgresql_python_library }}"
  state: present
```

Dans vars creer CentOS-7.yml
```shell
---
postgresql_version: "9.2"
postgresql_data_dir: "/var/lib/pgsql/data"
postgresql_bin_path: "/usr/bin"
postgresql_config_path: "/var/lib/pgsql/data"
postgresql_daemon: postgresql
postgresql_packages:
- postgresql
- postgresql-server
- postgresql-contrib
- postgresql-libs
  postgresql_python_library:
- postgresql-plpython
- python-psycopg2
```










## Docker-compose ou ansible-playbook 
Connectez vous en ssh sur le remote 
Deployez todo-flask-postgres 
installez docker
installez ansible, docker, docker-compose













































