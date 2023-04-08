# Ansible Resources

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=4 orderedList=false} -->

<!-- code_chunk_output -->

- [Ansible Resources](#ansible-resources)
  - [Documentation](#documentation)
  - [Labs](#labs)
  - [Tools](#tools)
    - [Quick Setup Lab](#quick-setup-lab)
  - [Index](#index)
    - [Ansible Commands List](#ansible-commands-list)
    - [Ansible Structure](#ansible-structure)
      - [Ansible Configuration](#ansible-configuration)
      - [Precedence Rules](#precedence-rules)
    - [Playbook Keywords](#playbook-keywords)
      - [Play Keywords](#play-keywords)
      - [Block Keywords](#block-keywords)
      - [Task Keywords](#task-keywords)
    - [Ansible Modules And Plugins List](#ansible-modules-and-plugins-list)
      - [General Modules](#general-modules)
      - [Role Modules](#role-modules)
      - [File Modules](#file-modules)
      - [Security Modules](#security-modules)
      - [Software Package Modules](#software-package-modules)
      - [System Modules](#system-modules)
      - [Users And Authentication Modules](#users-and-authentication-modules)
      - [Storage and Filesystems Modules](#storage-and-filesystems-modules)
      - [Net Tools Modules](#net-tools-modules)

<!-- /code_chunk_output -->

---

## Quick Setup Lab

generate

    ssh-keygen -t rsa

end control node ssh key to managed nodes

    ssh-copy-id user@node

configure NOPASSWD for user in managed nodes

    # replace inventory, node, user if necessary
    ansible -i inventory node -u root -m lineinfile -a 'create=true state=present path=/etc/sudoers.d/user owner=root group=root mode=`0644\' line="user ALL=(ALL) NOPASSWD: ALL"'

configure vimrc

    vim ~/.vimrc
      autocmd filetype yml,yaml setlocal et ai nu ts=2 sw=2 sts=2

---

## Index

Documentation:
https://docs.ansible.com/ansible/latest/
https://galaxy.ansible.com/docs/

Links:
https://galaxy.ansible.com/home
https://github.com/linux-system-roles

---

### Ansible Commands List

**Setup And Configuration Commands**

    echo "autocmd FileType yml,yaml setlocal et ai ts=2 sw=2" >> ~/.vimrc
    echo "autocmd FileType yml,yaml setlocal et ai nu ts=2 sw=2  sts=2" >> ~/.vimrc

    ansible-config dump
    ansible-config view
    ansible-config list

    ansible-doc --type plugin_type --list
    ansible-doc --type plugin_type --snippet [plugin [plugin]]

**Basic/Adhoc Commands**

    ansible --list-hosts host.example.com
    ansible --list-hosts group_example
    ansible-inventory --list -i inventory

    ansible all -m setup -a 'filter=*devices' --tree /tree/path
    ansible all -m setup -a 'gather_subset=!all,!min,network'
    ansible host.example.com -a "ls -l /tmp/"
    ansible host.example.com -m ping --become
    ansible host.example.com -m command -a 'df -h'

**Vault Commands**

    ansible-vault view    secret.yml
    ansible-vault edit    secret.yml
    ansible-vault create  secret.yml
    ansible-vault encrypt vars.yml --output=secret.yml
    ansible-vault decrypt secret.yml --output=vars.yml
    ansible-vault rekey   secret.yml --new-vault-password-file=rekeyfile

    ansible-playbook playbook.yml --ask-vault-pass
    ansible-playbook playbook.yml --vault-id label@source
    ansible-playbook playbook.yml --vault-id [default]@prompt
    ansible-playbook playbook.yml --vault-id id_label_2@prompt --vault-id label_2@passfile
    ansible-playbook playbook.yml --vault-password-file=pass file

**Playbook Commands**

    ansible-playbook playbook.yml -e "package=apache"
    ansible-playbook playbook.yml -i inventory -f fork_number
    ansible-playbook playbook.yml --list-tasks
    ansible-playbook playbook.yml --syntax-check
    ansible-playbook playbook.yml --check
    ansible-playbook playbook.yml --check --diff
    ansible-playbook playbook.yml --step
    ansible-playbook playbook.yml --start-at-task 'start httpd service'

**Galaxy Commands**

    ansible-galaxy list
    ansible-galaxy search 'search_term' --platforms EL --galaxy-tags
    ansible-galaxy search --author geerlingguy
    ansible-galaxy install geerlingguy.redis -p roles/
    ansible-galaxy install -r roles/requirements.yml -p roles
    ansible-galaxy init my_new_role
    ansible-galaxy remove rolename

---

### Ansible Structure

#### Ansible Configuration

Config precedence by order:

    1. $ANSIBLE_CONFIG    environment variable containing path
    2. ansible.cfg       in current directory
    3. ~/.ansible.cfg
    4. /etc/ansible/ansible.cfg

**ansible.cfg**

    [defaults]
    inventory=/etc/ansible/hosts
    roles_path=~/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles
    remote_user=user
    ask_pass=False
    gathering=implicit
    forks=5
    inject_facts_as_vars=True
    ansible_managed=Ansible managed

    [privilege_escalation]
    become=True
    become_method=sudo
    become_user=root
    become_ask_pass=False

#### Precedence Rules

**Variables Precedence**

    command line values (for example, -u my_user, these are not variables)
    role defaults (defined in role/defaults/main.yml)
    inventory file or script group vars
    inventory group_vars/all
    playbook group_vars/all
    inventory group_vars/*
    playbook group_vars/*
    inventory file or script host vars
    inventory host_vars/*
    playbook host_vars/*
    host facts / cached set_facts
    play vars
    play vars_prompt
    play vars_files
    role vars (defined in role/vars/main.yml)
    block vars (only for tasks in block)
    task vars (only for the task)
    include_vars
    set_facts / registered vars
    role (and include_role) params
    include params
    extra vars (for example, -e "user=my_user")(always win precedence)

Documentation:
https://docs.ansible.com/ansible/latest/reference_appendices/general_precedence.html#controlling-how-ansible-behaves-precedence-rules

Documentation:
https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable

---

### Playbook Keywords

Documentation:
https://docs.ansible.com/ansible/latest/reference_appendices/playbooks_keywords.html

Categories for playbook keywords:

- **Play**
- **Role**
- **Block**
- **Task**

#### Play Keywords

- name
- hosts
- force_handlers
- gather_facts
- gather_subset
- serial

**Privilege Escalation**

- remote_user
- become
- become_method
- become_user

**Variables**

- vars
- vars_files
- vars_prompt

**Tasks Sections**

- pre_tasks
- roles
- tasks
- post_tasks
- handlers

#### Block Keywords

- block
- rescue
- always
- name
- vars
- when

**Privilege Escalation**

- remote_user
- become
- become_method
- become_user

#### Task Keywords

**General**

- name
- module_name
- register
- vars

**Privilege Escalation**

- remote_user
- become
- become_method
- become_user

**Loops And Conditionals**

- when
- run_once
- loop
- until + retries + delay
- with_items
- with_file
- with_sequence

**Handling Failures**

- notify
- changed_when
- failed_when
- ignore_errors
- ignore_unreachable

---

### Ansible Modules And Plugins List

Documentation:
https://docs.ansible.com/ansible/latest/collections/index_module.html#ansible-builtin

#### General Modules

- raw
- shell
- command
- ping
- debug
- fail

#### Role Modules

- include_role
- import_role

#### File Modules

- stat
- file
  Params: path(req), state, owner, group, mode
- lineinfile
- blockinfile
- archive
- unarchive

- copy
  Params: src(req), dest(req), owner, group, mode
- fetch
- template
- synchronize

#### Security Modules

- selinux
- seboolean
  target requires packages: python3 libsemanage-python3
- sefcontext
  target requires packages: policycoreutils-python3

#### Software Package Modules

Redhat Subscription

- redhat_subscription
- rhsm_repository

Yum Repositories

- yum_repository
- rpm_key

Packages

- package
- package-facts
- dnf
  Params: name, state
- yum
- apt

- pip
- git
  Params: repo(req), dest(req), clone

#### System Modules

- at
- cron

- reboot
- systemd
- service
  Params: name(req), state, enabled

#### Users And Authentication Modules

- user
  Params: name(req), state, uid, group, groups, remove, shell, home, password, ...
- group
- known_hosts
- authorized_key

#### Storage and Filesystems Modules

- parted
- lvg
- lvol
- filesystem
- mount
  states: (+ added; x removed; - not changed)
  mounted (mount: +; fstab: +)
  absent (mount: x; fstab: x)
  present (mount: -; fstab: +)
  unmounted (mount: x; fstab: -)

#### Net Tools Modules

- nmcli
- hostname
- firewalld
- get_url: Download files over HTTP, HTTPS, or FTP
- uri: Interact with web services

**Plugins**

Documentation:
https://docs.ansible.com/ansible/latest/plugins/plugins.html
https://docs.ansible.com/ansible/latest/collections/index_lookup.html

Augmentations to the control node system, used to reference modules in documentation. Plug-in types:

    become, cache, callback, cliconf, connection, httpapi,
    inventory, lookup, netconf, shell, module, strategy,
    vars

- lookup

  allows Ansible to access data from outside sources
  https://docs.ansible.com/ansible/latest/plugins/lookup.html?highlight=lookup
