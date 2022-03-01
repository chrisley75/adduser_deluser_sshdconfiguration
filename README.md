# Manage Authentication and SSH Configuration

These ansible playbooks will manage to create users with associated public ssh key in input and SSH configuration. Security rules will also be configured and deployed.

Main uses of this playbook:
- Create new default user "ops-user" to manage operations
- Create new user groups "admin" and "ops"
- Create a new personal user with an associated public key 
- Configure SSHD to comply with security policies 
- Delete old users
- Manage sudoers rules

# Requirement
- Ansible version 2.9.x and above

# Prerequisites
- User with a privileged account that will be used by ansible to connect to the servers
- User Public Key must be provided in input 

# Groups

By default two user groups are created:
- admin : grant to have elevated sudoer rules
- ops : all ops users


Only Administrator are in admin group.

NB: It is possible to create new groups and add user to group with this playbook

# Playbooks

- playbook_ssh_add_users.yml : add new user + key and configure sshd 
- playbook_remove_users.yml : delete and disable old users
- playbook_restart_sshd.yml : restart SSH daemon


# hosts / inventory

Edit hosts file to add target server.

Example hosts file:
```bash
[server]
10.0.0.1

[server2]
10.0.0.2
```

Use hosts variable into playbook as target:

hosts: "{{ target | default('nothing') }}" allows you to specify as a parameter of your ansible-playbook command on which group or even on a specific machine you want to launch your playbook, the syntax is as follows:



For example to launch on the server the command is :
```bash
ansible-playbook -i hosts -e target="server" playbook.yml
```

For example to launch on the server2 the command is :
```bash
ansible-playbook -i hosts -e target="server2" playbook.yml 
```

Or another example if you want to target just one machine, below example for target server with IP 10.0.0.10, (no need to declare the ip in the hosts file) :
```bash
ansible-playbook -i hosts -e target="10.0.0.10" playbook.yml 
```

# Create New Users (playbook_ssh_add_users.yml)

1) Add ssh public key for user into public_keys directory

Name of the public_key should match the name of the user in vars section

Naming convention:
<firstname>-<name>.key.pub

Example:
christopher-ley.key.pub

```bash
public_keys/
└── christopher-ley.key.pub
```

2) Edit vars.users section in playbook_ssh_add_users.yml

```bash
  vars:
    users:
      - username: "<firstname>-<name>"
        groups: "<group>"
```

Example for adding user christopher-ley in groups cms and admin:

```bash
  vars:
    users:
      - username: "christopher-ley"
        groups: "admin,ops"
```

Usage: 
```bash
$ ansible-playbook playbook_ssh_add_users.yml -i hosts -e target="server" --user <privileged user> --private-key=<private_key>  | or use --user "root" --ask-pass if first run

PLAY [server] ****************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************************
ok: [xxx]

TASK [ssh : Disable root login via SSH] *************************************************************************************************************************************************************************
ok: [xxx]

TASK [ssh : Disable Password Authentification via SSH] **********************************************************************************************************************************************************
ok: [xxx]

TASK [add_users : Create user groups] ***************************************************************************************************************************************************************************
ok: [xxx] => (item=admin)
ok: [xxx] => (item=ops)

TASK [add_users : Create ops-user account] *******************************************************************************************************************************************************************
ok: [xxx]

TASK [add_users : Create user accounts and add users to groups] *************************************************************************************************************************************************
ok: [xxx] => (item={'username': 'christopher-ley', 'groups': 'admin,ops'})

TASK [add_users : Add authorized keys] **************************************************************************************************************************************************************************
ok: [xxx] => (item={'username': 'christopher-ley', 'groups': 'admin,ops'})

PLAY RECAP ******************************************************************************************************************************************************************************************************
xxx    : ok=7    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

# Delete Old Users (playbook_remove_users.yml)

1) Edit vars.remove_users section in playbook_remove_users.yml

```bash
  vars:
    remove_user:
      - <user>
```

Example for removing user tester

```bash
  vars:
    remove_user:
      - tester
```
Usage:
```bash
$ ansible-playbook playbook_remove_users.yml -i hosts -e target="server" --user christopher-ley --private-key=/home/christopher/.ssh/cley_id_rsa

PLAY [server] ****************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************************
ok: [xxx]

TASK [remove_users : Remove user accounts] **********************************************************************************************************************************************************************
changed: [xxx] => (item=tester)

PLAY RECAP ******************************************************************************************************************************************************************************************************
xxx    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

# SSH Configuration

This playbook will apply an SSH configuration in accordance with the security rules.
Contact the security administrators to find out what these rules are.

ATTENTION: New SSH configurations will be applied.
This will impact the default SSH access currently defined on servers.

Reload SSH to apply new configuration by using playbook_restart_sshd.yml
```bash
$ ansible-playbook playbook_restart_sshd.yml -i hosts -e target="server" --user christopher-ley --private-key=/home/christopher/.ssh/cley_id_rsa

PLAY [nexus] ****************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************************
ok: [xxx]

TASK [Restart SSHD to apply modification] ***********************************************************************************************************************************************************************
changed: [xxx]

PLAY RECAP ******************************************************************************************************************************************************************************************************
xxx    : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

# Usage

Day0 : First run with default user (ie root):
```bash
$ ansible-playbook playbook.yml -i hosts -e target="server" --user root --ask-pass
```
you will be asked to fill in the root password.
This feature will be remove after running securities playbook (password authentication disabled)

Day1 : Common run with privileged user and sshkey
```bash
$ ansible-playbook playbook.yml -i hosts -e target="server" --user christopher-ley --private-key=/home/christopher/.ssh/cley_id_rsa
```

