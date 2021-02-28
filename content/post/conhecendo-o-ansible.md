---
author: "João Maia"
date: 2020-07-24
title: "Conhecendo o Ansible"
tags: [
    "ansible",
]
categories: [
    "devops"
]
thumbnail: "img/conhecendo-o-ansible/ansible-logo.png"
---

Olá,

dando continuidade nos artigos de infraestrutura como código chegou a hora de escrever sobre o [Ansible](https://www.ansible.com/). Um pouco de história, o Ansible veio como solução semelhante ao [Puppet](https://puppet.com/) e [Chef](https://www.chef.io/) pela empresa de mesmo nome com o produto sendo o [Ansible Tower](https://www.ansible.com/products/tower) que depois foi comprada pela [Red Hat](https://www.redhat.com/pt-br) e agora a Red Hat foi comprada pela [IBM](https://www.ibm.com/br-pt). Meu primeiro contato foi em 2017 depois de uma experiência bem simples com o Puppet. Achei o Ansible muito mais intuitivo ao processo que tinha sempre feito na vida de dar ssh no servidor e rodar scripts em Bash ou fazer manualmente vários processos, vi no Ansible uma forma simples e direta de fazer isso.

> Ansible is an open source community project sponsored by Red Hat, it's the simplest way to automate IT. Ansible is the only automation language that can be used across entire IT teams from systems and network administrators to developers and managers.

# Instalando o Ansible

A [página oficial](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) de instalação do Ansible é bem completa, eu usei por muito tempo o modo via [pip](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#installing-ansible-with-pip) mas hoje tenho instalado via [Linux Brew](https://brew.sh/). Se você tiver necessidade de trabalhar com versões muito específicas do Ansible acredito que a opção do `pip` vai ser a melhor para o seu caso.

# Como o Ansible funciona

O Ansible é escrito em [Python](https://www.python.org/) e um requisito dele é o [Paramiko](http://www.paramiko.org/) que é uma biblioteca que implementa o protocolo SSHv2. Saber disso esclarece bastante sobre o seu funcionamento pois é através do SSH que o Ansible se conecta nas máquinas que vai provisionar para executar suas tarefas. Então, é muito importante que ao montar o seu playbook você tenha em mente como vai conseguir acessar essa máquinas, imagina que sua receita de provisionamento tem que configurar 100 máquinas, você não querer ficar digitando senha 100 vezes, né? Na AWS a solução mais tradicional para acesso SSH as máquinas são as chaves PEM, veja [aqui](https://docs.aws.amazon.com/pt_br/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html) mais sobre, existem outra formas mas nesse artigo vamos trabalhar com essa apenas.

# Estrutura de um projeto Ansible

No [guia de boas práticas do Ansible](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html#directory-layout) são apresentadas duas abordagens de organização do seu projeto:

```yaml
production                # inventory file for production servers
staging                   # inventory file for staging environment

group_vars/
   group1.yml             # here we assign variables to particular groups
   group2.yml
host_vars/
   hostname1.yml          # here we assign variables to particular systems
   hostname2.yml

library/                  # if any custom modules, put them here (optional)
module_utils/             # if any custom module_utils to support modules, put them here (optional)
filter_plugins/           # if any custom filter plugins, put them here (optional)

site.yml                  # master playbook
webservers.yml            # playbook for webserver tier
dbservers.yml             # playbook for dbserver tier

roles/
    common/               # this hierarchy represents a "role"
        tasks/            #
            main.yml      #  <-- tasks file can include smaller files if warranted
        handlers/         #
            main.yml      #  <-- handlers file
        templates/        #  <-- files for use with the template resource
            ntp.conf.j2   #  <------- templates end in .j2
        files/            #
            bar.txt       #  <-- files for use with the copy resource
            foo.sh        #  <-- script files for use with the script resource
        vars/             #
            main.yml      #  <-- variables associated with this role
        defaults/         #
            main.yml      #  <-- default lower priority variables for this role
        meta/             #
            main.yml      #  <-- role dependencies
        library/          # roles can also include custom modules
        module_utils/     # roles can also include custom module_utils
        lookup_plugins/   # or other types of plugins, like lookup in this case

    webtier/              # same kind of structure as "common" was above, done for the webtier role
    monitoring/           # ""
    fooapp/               # ""
```

e

```yaml
inventories/
   production/
      hosts               # inventory file for production servers
      group_vars/
         group1.yml       # here we assign variables to particular groups
         group2.yml
      host_vars/
         hostname1.yml    # here we assign variables to particular systems
         hostname2.yml

   staging/
      hosts               # inventory file for staging environment
      group_vars/
         group1.yml       # here we assign variables to particular groups
         group2.yml
      host_vars/
         stagehost1.yml   # here we assign variables to particular systems
         stagehost2.yml

library/
module_utils/
filter_plugins/

site.yml
webservers.yml
dbservers.yml

roles/
    common/
    webtier/
    monitoring/
    fooapp/
```

Acho as duas formas de organizar muito boas mas o que vai definir qual será usada será o seu projeto além do que existe a possibilidade de você trabalhar com [inventários dinâmicos](https://docs.ansible.com/ansible/latest/user_guide/intro_dynamic_inventory.html#intro-dynamic-inventory). O importante dessas duas abordagens é apresentar os elementos fundamentais do Ansible: `playbook`, `roles` e `inventory`. Estou ignorando nesse momento a parte de `plugins` do Ansible pois é melhor tratar isso em outro momento pois não é o uso mais comum. Vamos agora entender melhor os outros três elementos.

## Playbook

Esse é o coração do seu projeto, olhando nos exemplos acima de organização de projeto temos três: `site.yml`, `webservers.yml` e `dbservers.yml`. Vamos assumir que nessa estrutura dada o propósito era subir um blog usando o Wordpress, quem para quem não conhece o Wordpress é uma ferramenta para criar blogs escrita em PHP que vai precisar de um servidor de aplicações como o Apache ou NGINX e depende de um banco de dados, no caso o MySQL. Então, qual poderiam ser os objetivos de cada um desses playbooks? No `dbservers.yml` é onde vamos fazer todos os passos necessários para termos de forma independente o nosso banco de dados funcionando seja ele um banco de uma instância apenas com ou sem backup, tudo relativo a vida desse serviço fica descrita nesse playbook através de passos simples e bem claros como se fosse uma pessoa executando manualmente. No `webservers.yml` vamos colocar a combinação NGINX e Wordpress porém nesse caso o nosso módulo não consegue viver de forma independente pois ele precisa de uma banco de dados. Então, chegamos no `site.yml` que é o nosso projeto final que junta o Wordpress com o MySQL assim possibilitando seu blog de existir.

O que podemos entender aqui dos playbooks é que eles são roteiros de execução de comandos ou alterações no seu alvo que podem ser totalmente independentes ou dependentes de outros playbooks.

## Roles

Olhando novamente o exemplo acima temos dentro de roles a pasta `monitoring`. Seria interessante tanto no nosso Wordpress quanto o MySQL termos uma monitoria desses serviços e como é boa prática usar uma ferramenta de monitoria comum em toda sua arquitetura podemos criar um componente que configura essa monitoria de forma segregada do contexto principal do projeto e usar como biblioteca. Então, basicamente podemos entender que roles são outros projetos Ansible que tem toda a sua estrutra para construir partes de sua infraestrutura de forma configurável por outros projetos. Então, assim como o `pip` que é um gestor de bibliotecas em Python onde é distribuido o Ansible e suas dependências em Python, o Ansible tem uma ferramenta para tal, esse é o [Ansible Galaxy](https://galaxy.ansible.com/). Com o Ansible Galaxy você é capaz de gerenciar suas dependências de roles no projeto e em um sistema de comunidade colaborar ou receber ajuda de outras pessoas em projetos importantes para a sua infraestrutura.

## Inventory

Muito legal você ser capaz de executar toda essa automação de infraestrutura editando apenas alguns arquivos YAML. Só que para isso tudo funcionar é preciso de um alvo e é no inventory que vamos definir isso seja passando uma lista de máquinas ou apenas uma, como acessar essas máquinas, variáveis associadas a essas máquinas e é possível também agrupar algumas dessas máquinas em grupos, por exemplo, um cluster de uma banco de dados NoSQL provavelmente vai compartilhar muita informação e podem ser agrupadas em uma configuração apenas.

# Seu primeiro Playbook

Com essa introdução teórica do Ansible já podemos partir um pouco para o uso prático com exemplos, no meu Github tenho um [repositório](https://github.com/jvrmaia/ansible-playground) que usei para demonstrar um pouco do Ansible que vou aproveitar nesse artigo. Neste respositório eu usei o Vagrant para tester os playbooks do Ansible, caso não tenha Vagrant no seu ambiente é possível subir uma máquina na nuvem de sua preferência e fazer uns ajustes na parte de conectar na máquina ou usar Docker localmente, se precisar de ajuda com isso avise nos comentários. O exemplo que vamos trabalhar aqui é o da pasta `ansible-roles-and-playbook`.

Neste projeto a máquina alvo a ser provisionada é um CentOS 7 que pode ser conferido no Vagrantfile e vai ter um IP 20.20.20.20 que é acessível da minha máquina local e a chave de SSH para acesso vai ser a minha local do Vagrant, detalhe que eu não preciso fazer nada aqui além de rodar o Vagrant. Agora, vamos navegar pelos arquivos da pasta falando um pouco mais de cada.

## ansible.cfg

```toml
[defaults]
hostfile=hosts
roles_path=./roles

[ssh_connection]
ssh_args = -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no -o IdentitiesOnly=yes -o ControlMaster=auto -o ControlPersist=60s
```

Esse é o arquivo de configuração local do Ansible, como podem ver estou informando nele quel é o meu arquivo que contém as informações do hosts do projeto, onde estão minhas roles e algumas configurações do SSH que é a forma que o Ansible usa para se conectar nas máquinas.

## hosts

```toml 
[db]
20.20.20.20

[db:vars]
ansible_ssh_user=vagrant
ansible_ssh_private_key_file=~/.vagrant.d/insecure_private_key
```

Nesse projeto vamos subir um banco de dados apenas então criei essa entrada `[db]` para informar seus dados que nesse caso é apenas o seu IP que é o mesmo definido no Vagrant. Em `[db:vars]`, estou definindo umas variáveis referentes a essa máquina virtual que são seu usuário SSH e sua chave de acesso, por exemplo, quem optar por usar uma máquina na AWS com o Amazon Linux o valor do usuário SSH seria `ec2-user` e o valor da chave seria o caminho para sua chave PEM.

## requirements.yml

```yaml
---
- src: geerlingguy.mysql
```

Esse arquivo será usando pelo Ansible Galaxy para gerir suas dependências, no caso estamos usando uma role pronta do MySQL pois não quero lidar com isso e prefiro a delegar para alguém que já fez que confio e toda a comunidade que já colaborou com ela. Para ter as roles no seu ambiente local para usar é bem simples:

```
$ ansible-galaxy install -r requirements.yml 
- downloading role 'mysql', owned by geerlingguy
- downloading role from https://github.com/geerlingguy/ansible-role-mysql/archive/3.1.0.tar.gz
- extracting geerlingguy.mysql to /opt/roles/geerlingguy.mysql
- geerlingguy.mysql (3.1.0) was installed successfully
```

## test.yml

```yaml
---
- name: Debug date
  hosts: localhost
  tasks:
    - command: date
      register: date

    - debug: var=date
```

Esse é um exemplo de playbook que alterei da versão do Github na parte de `hosts` para apontar para minha máquina local apenas. Podemos ver que a estrutura é bem simples, primeiro é dado um nome ao conjunto de tarefas que vamos executar, definimos o alvo e por fim uma lista de tarefas. Nas tarefas o que fiz foi apenas executar um comando do Linux, `date`, e salvar o seu valor na variável `date`. Por fim, uso a opção de `debug` do Ansible para imprimir o valor retornado e outras informações mais no terminal. Veja o resultado da execução do playbook:

```bash
$ ansible-playbook test.yml 
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match 'all'

PLAY [Debug date] ********************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [command] ***********************************************************************************************************************************************************************************************************************************************
changed: [localhost]

TASK [debug] *************************************************************************************************************************************************************************************************************************************************
ok: [localhost] => {
    "date": {
        "changed": true,
        "cmd": [
            "date"
        ],
        "delta": "0:00:00.002301",
        "end": "2020-07-22 06:23:53.943966",
        "failed": false,
        "rc": 0,
        "start": "2020-07-22 06:23:53.941665",
        "stderr": "",
        "stderr_lines": [],
        "stdout": "Wed Jul 22 06:23:53 -03 2020",
        "stdout_lines": [
            "Wed Jul 22 06:23:53 -03 2020"
        ]
    }
}

PLAY RECAP ***************************************************************************************************************************************************************************************************************************************************
localhost                  : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
  
```

## provision.yml

```yaml
---
- name: Configure database server
  hosts: db
  become: yes
  vars:
    - mysql_root_password: super-secure-password
    - mysql_databases:
      - name: my_app_db
        encoding: utf8
        collation: utf8_general_ci
    - mysql_users:
      - name: app_user
        host: "%"
        password: similarly-secure-password
        priv: "my_app_db.*:ALL"
  roles:
    - { role: geerlingguy.mysql }
```

Mudei um pouco do original pois movi o `become` para o playbook inteiro e não apenas a role e mudei as configurações do host para ser uma instância CentOS 8 que subi na AWS. Como podem ver é bem parecido com o playbook de teste mudando que não tem tarefas nesse, tudo ficou contido na role precisando apenas passar algumas variáveis para ele funcionar. Vamos executar para ver a execução dele:

```bash
$ ansible-playbook provision.yml -i hosts 

PLAY [Configure database server] *****************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***************************************************************************************************************************************************************************************************************************************
ok: [18.222.238.109]

TASK [geerlingguy.mysql : include_tasks] *********************************************************************************************************************************************************************************************************************
included: /home/jvrmaia/Workspace/jvrmaia/ansible-playground/ansible-roles-and-playbook/roles/geerlingguy.mysql/tasks/variables.yml for 18.222.238.109

TASK [geerlingguy.mysql : Include OS-specific variables.] ****************************************************************************************************************************************************************************************************
ok: [18.222.238.109] => (item=/home/jvrmaia/Workspace/jvrmaia/ansible-playground/ansible-roles-and-playbook/roles/geerlingguy.mysql/vars/RedHat-8.yml)

TASK [geerlingguy.mysql : Define mysql_packages.] ************************************************************************************************************************************************************************************************************
ok: [18.222.238.109]

TASK [geerlingguy.mysql : Define mysql_daemon.] **************************************************************************************************************************************************************************************************************
ok: [18.222.238.109]

TASK [geerlingguy.mysql : Define mysql_slow_query_log_file.] *************************************************************************************************************************************************************************************************
ok: [18.222.238.109]

TASK [geerlingguy.mysql : Define mysql_log_error.] ***********************************************************************************************************************************************************************************************************
ok: [18.222.238.109]

TASK [geerlingguy.mysql : Define mysql_syslog_tag.] **********************************************************************************************************************************************************************************************************
ok: [18.222.238.109]

TASK [geerlingguy.mysql : Define mysql_pid_file.] ************************************************************************************************************************************************************************************************************
ok: [18.222.238.109]

TASK [geerlingguy.mysql : Define mysql_config_file.] *********************************************************************************************************************************************************************************************************
ok: [18.222.238.109]

TASK [geerlingguy.mysql : Define mysql_config_include_dir.] **************************************************************************************************************************************************************************************************
ok: [18.222.238.109]

TASK [geerlingguy.mysql : Define mysql_socket.] **************************************************************************************************************************************************************************************************************
ok: [18.222.238.109]

TASK [geerlingguy.mysql : Define mysql_supports_innodb_large_prefix.] ****************************************************************************************************************************************************************************************
ok: [18.222.238.109]

TASK [geerlingguy.mysql : include_tasks] *********************************************************************************************************************************************************************************************************************
included: /home/jvrmaia/Workspace/jvrmaia/ansible-playground/ansible-roles-and-playbook/roles/geerlingguy.mysql/tasks/setup-RedHat.yml for 18.222.238.109

TASK [geerlingguy.mysql : Ensure MySQL packages are installed.] **********************************************************************************************************************************************************************************************
changed: [18.222.238.109]

TASK [geerlingguy.mysql : include_tasks] *********************************************************************************************************************************************************************************************************************
skipping: [18.222.238.109]

TASK [geerlingguy.mysql : include_tasks] *********************************************************************************************************************************************************************************************************************
skipping: [18.222.238.109]

TASK [geerlingguy.mysql : Check if MySQL packages were installed.] *******************************************************************************************************************************************************************************************
ok: [18.222.238.109]

TASK [geerlingguy.mysql : include_tasks] *********************************************************************************************************************************************************************************************************************
included: /home/jvrmaia/Workspace/jvrmaia/ansible-playground/ansible-roles-and-playbook/roles/geerlingguy.mysql/tasks/configure.yml for 18.222.238.109

TASK [geerlingguy.mysql : Get MySQL version.] ****************************************************************************************************************************************************************************************************************
ok: [18.222.238.109]

TASK [geerlingguy.mysql : Copy my.cnf global MySQL configuration.] *******************************************************************************************************************************************************************************************
changed: [18.222.238.109]

TASK [geerlingguy.mysql : Verify mysql include directory exists.] ********************************************************************************************************************************************************************************************
skipping: [18.222.238.109]

TASK [geerlingguy.mysql : Copy my.cnf override files into include directory.] ********************************************************************************************************************************************************************************

TASK [geerlingguy.mysql : Create slow query log file (if configured).] ***************************************************************************************************************************************************************************************
skipping: [18.222.238.109]

TASK [geerlingguy.mysql : Create datadir if it does not exist] ***********************************************************************************************************************************************************************************************
ok: [18.222.238.109]

TASK [geerlingguy.mysql : Set ownership on slow query log file (if configured).] *****************************************************************************************************************************************************************************
skipping: [18.222.238.109]

TASK [geerlingguy.mysql : Create error log file (if configured).] ********************************************************************************************************************************************************************************************
skipping: [18.222.238.109]

TASK [geerlingguy.mysql : Set ownership on error log file (if configured).] **********************************************************************************************************************************************************************************
skipping: [18.222.238.109]

TASK [geerlingguy.mysql : Ensure MySQL is started and enabled on boot.] **************************************************************************************************************************************************************************************
changed: [18.222.238.109]

TASK [geerlingguy.mysql : include_tasks] *********************************************************************************************************************************************************************************************************************
included: /home/jvrmaia/Workspace/jvrmaia/ansible-playground/ansible-roles-and-playbook/roles/geerlingguy.mysql/tasks/secure-installation.yml for 18.222.238.109

TASK [geerlingguy.mysql : Ensure default user is present.] ***************************************************************************************************************************************************************************************************
skipping: [18.222.238.109]

TASK [geerlingguy.mysql : Copy user-my.cnf file with password credentials.] **********************************************************************************************************************************************************************************
skipping: [18.222.238.109]

TASK [geerlingguy.mysql : Disallow root login remotely] ******************************************************************************************************************************************************************************************************
ok: [18.222.238.109] => (item=DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1'))

TASK [geerlingguy.mysql : Get list of hosts for the root user.] **********************************************************************************************************************************************************************************************
ok: [18.222.238.109]

TASK [geerlingguy.mysql : Update MySQL root password for localhost root account (5.7.x).] ********************************************************************************************************************************************************************
skipping: [18.222.238.109] => (item=127.0.0.1) 
skipping: [18.222.238.109] => (item=::1) 
skipping: [18.222.238.109] => (item=localhost) 

TASK [geerlingguy.mysql : Update MySQL root password for localhost root account (< 5.7.x).] ******************************************************************************************************************************************************************
changed: [18.222.238.109] => (item=127.0.0.1)
changed: [18.222.238.109] => (item=::1)
changed: [18.222.238.109] => (item=localhost)

TASK [geerlingguy.mysql : Copy .my.cnf file with root password credentials.] *********************************************************************************************************************************************************************************
changed: [18.222.238.109]

TASK [geerlingguy.mysql : Get list of hosts for the anonymous user.] *****************************************************************************************************************************************************************************************
ok: [18.222.238.109]

TASK [geerlingguy.mysql : Remove anonymous MySQL users.] *****************************************************************************************************************************************************************************************************

TASK [geerlingguy.mysql : Remove MySQL test database.] *******************************************************************************************************************************************************************************************************
ok: [18.222.238.109]

TASK [geerlingguy.mysql : include_tasks] *********************************************************************************************************************************************************************************************************************
included: /home/jvrmaia/Workspace/jvrmaia/ansible-playground/ansible-roles-and-playbook/roles/geerlingguy.mysql/tasks/databases.yml for 18.222.238.109

TASK [geerlingguy.mysql : Ensure MySQL databases are present.] ***********************************************************************************************************************************************************************************************
changed: [18.222.238.109] => (item={'name': 'my_app_db', 'encoding': 'utf8', 'collation': 'utf8_general_ci'})

TASK [geerlingguy.mysql : include_tasks] *********************************************************************************************************************************************************************************************************************
included: /home/jvrmaia/Workspace/jvrmaia/ansible-playground/ansible-roles-and-playbook/roles/geerlingguy.mysql/tasks/users.yml for 18.222.238.109

TASK [geerlingguy.mysql : Ensure MySQL users are present.] ***************************************************************************************************************************************************************************************************
changed: [18.222.238.109] => (item=None)
changed: [18.222.238.109]

TASK [geerlingguy.mysql : include_tasks] *********************************************************************************************************************************************************************************************************************
included: /home/jvrmaia/Workspace/jvrmaia/ansible-playground/ansible-roles-and-playbook/roles/geerlingguy.mysql/tasks/replication.yml for 18.222.238.109

TASK [geerlingguy.mysql : Ensure replication user exists on master.] *****************************************************************************************************************************************************************************************
skipping: [18.222.238.109]

TASK [geerlingguy.mysql : Check slave replication status.] ***************************************************************************************************************************************************************************************************
skipping: [18.222.238.109]

TASK [geerlingguy.mysql : Check master replication status.] **************************************************************************************************************************************************************************************************
skipping: [18.222.238.109]

TASK [geerlingguy.mysql : Configure replication on the slave.] ***********************************************************************************************************************************************************************************************
skipping: [18.222.238.109]

TASK [geerlingguy.mysql : Start replication.] ****************************************************************************************************************************************************************************************************************
skipping: [18.222.238.109]

RUNNING HANDLER [geerlingguy.mysql : restart mysql] **********************************************************************************************************************************************************************************************************
[WARNING]: Ignoring "sleep" as it is not used in "systemd"
changed: [18.222.238.109]

PLAY RECAP ***************************************************************************************************************************************************************************************************************************************************
18.222.238.109             : ok=34   changed=8    unreachable=0    failed=0    skipped=17   rescued=0    ignored=0   
```

Repare pelo log como é feito o passo a passo para configurar o MySQL desde a identificação da máquina alvo do provisionamento. Imagina ter que realizar isso tudo de forma manual e as possibilidades de errar coisas triviais? Não ache também que isso tudo que mostrei é perfeito. Fiz na AWS e com CentOS porque no meu teste inicial com Docker e Ubuntu deu erro com o Python e pelo que vi não estava bem resolvido na versão que estou usando.

# Dicas gerais

- Sempre um `ansible.cfg` nos seus projetos para facilitar algumas configurações;
- Abuse dos `-v` ao rodar seus playbooks quando precisar depurar algum problema;
- Cuidado com as versõs de Python e dependências de pacotes do Python;
- Use a abuse de roles prontas no Ansible Galaxy;
- Evite complexidade no seu playbook principal, quebre em roles;
- Use um editor que tenha bom suporte para YAML; 
- Playbooks podem ser demorados para executar, se possível rode de uma VM dentro da sua infra para evitar problemas de rede e use screen ou tmux; e
- Não complique o simples.

# Referências para estudo

- [Canal da LinuxTips](https://www.youtube.com/user/linuxtipscanal)
- [Livros do Jeff Geerling na Leanpub](https://leanpub.com/u/geerlingguy)
- [O Matheus Fidelis está fazendo uma sequência de videos usando Ansible para subir uma stack CNCF](https://www.youtube.com/channel/UC-JJ1NXjx8QI-tuG0PMFbhA)
- [Recursos no site do Ansible](https://www.ansible.com/resources)

# Considerações finais

Ficou bem superficial mas era só para dar um "cheiro" no assunto mesmo pois vou precisar dessa pequena base de conhecimento para voltarmos naquele artigo do Packer e adicionar o Ansible nele, ainda vamos ter um de Terraform no meio do caminho.