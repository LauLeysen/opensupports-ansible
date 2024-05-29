
# OpenSupports Ansible Playbook

This repository contains an Ansible playbook for setting up OpenSupports on a target host. Note you will need a freshly installed rocky 9.4 linux with internet access also some parts of the code are altered to make it work since OpenSupports hasn't been updated since 2022. The following versions are mandatory for the installation:

- PHP 7.x+ (PHP 8 not supported)
- MySQL 4.1+
- PDO driver

Also note the database only works over https connection so the SSL is REALLY necessary and you have to use https://

## Prerequisites

Ensure you have the following installed on your control machine:

- Ansible
- SSH access to the target host
- Email setup with smtp I used outlook
    - smtp.office365.com:587

## Variables

Define your variables in `vars/main.yml`:

```yaml
# Variables for the playbook
---
opensupports_ip: xxx.xxx.xxx.xxx
opensupports_target_user: root
mysql_password: desiredsqlpassword
```

## SSH Key Transfer

You can easily send your SSH key from the host to the target host with the following command:

```sh
cat ~/.ssh/id_rsa.pub | ssh root@xxx.xxx.xxx.xxx 'mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys'
```

## Run the playbook

All you need to do is run this command in the directory!

```bash
ansible-playbook main.yml
```

## Project Timesheet

| Date       | Time Spent |
|------------|-------------|
| 26 June    | 4:24 hours  |
| 27 June    | 9:15 hours  |
| 28 June    | 3:10 hours  |
| 29 June    | 1:00 hours  |

## Playbook Structure

The main playbook (`main.yml`) looks like this:

```yaml
---
- name: Add target host to inventory
  hosts: localhost
  gather_facts: false
  vars_files:
    - vars/main.yml
  tasks:
    - name: Add target host
      add_host:
        name: "{{ opensupports_ip }}"
        ansible_user: "{{ opensupports_target_user }}"

- name: Run tasks on target host
  hosts: "{{ opensupports_ip }}"
  gather_facts: false
  vars_files:
    - vars/main.yml
  tasks:
    - include_tasks: tasks/info.yml
    - include_tasks: tasks/install_packages.yml
    - include_tasks: tasks/mysql.yml
    - include_tasks: tasks/opensupports.yml
    - include_tasks: tasks/cert.yml
```

### Task Files

- **info.yml**: Gathers facts about the system and shows the distribution version.
- **install_packages.yml**: Ensures the system is up to date, installs required packages, and sets up PHP 7.4.
- **mysql.yml**: Configures MySQL, sets the root password, and removes unnecessary users and databases.
- **opensupports.yml**: Sets up Apache, installs OpenSupports, configures permissions, and updates the configuration.
- **cert.yml**: Creates a self-signed SSL certificate, configures Apache to use it, and sets up a cron job to renew the certificate.

## Acknowledgments

Special thanks to the following resources:

- [Database setup](https://stackoverflow.com/questions/41645309/mysql-error-access-denied-for-user-rootlocalhost)
- [OpenSupports installation](https://docs.opensupports.com/guides/installation/)
- [ChatGPT: help with debugging some actions not working properly](https://openai.com/chatgpt)
- [White page issue](https://github.com/opensupports/opensupports/issues/1231)
- [PHP 7.4 installation](https://idroot.us/install-php-7-4-centos-stream-9/)
- [Read/Write issue for API](https://stackoverflow.com/questions/29343809/php-is-writable-function-always-returns-false-for-a-writable-directory/45071223#45071223)
- [SSL certificate setup](https://stackoverflow.com/questions/56350113/ansible-create-a-self-signed-ssl-certificate-and-key)
- [ChatGPT: generating some Ansible actions and helping debug some errors](https://openai.com/chatgpt)

Thank you for using this playbook!
