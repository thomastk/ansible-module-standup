# Introduction
An Ansible module to verify newly provisioned infrastructure and monitor its state, with the option to heal a state.

# Installation
Drop the code module, standup.py, under a directory "library".

# Ansible Documentation

This is copied from the code file standup.py and refer that for latest version.

```
module: standup
short_description: Runs a suite of checks to validate a host and heals state if needed.
description:
    - "The standup moudle runs a set of commands and evaluate the output to determine if a host stands up right. The tests can be specified in YAML format. A check contains a command that is supported on the host and directives to evaluate its output. An optional command to heal a state can also be specified with every check."
version_added: "2.3"
options:
    checks_file_path:
        description:
            - Path to the YAML file in which the checks are defined.
        required: true
    checks_format:
        description:
            - The format of checks_file_path. 
        required: false
        choices: ['yaml']
        default: yaml
    roles:
        description:
            - Host roles against which the validation has to be executed. If roles are specified, only the compatible checks from checks_file_path will be used for validation.
        required: false
    heal_state:
        description:
            - Indicates that heal action be performed if a check fails. The command to heal broken state must be available.
        required: false
        choices: [true, false]
        default: false
    continue_on_failure:
        description:
            - Indicates that the validation process continues in the event a check fails. A check status can be ignored at the check level also.
        required: false
        choices: [true, false]
        default: true
notes:
    - Tested on Mac, Ubuntu, RHEL and CentOS.
    - Run checks specified in an input file depending on the roles specified.
    - If the check command fails or the output doesn't get validated, an optional heal step can be run.
    - The state of host is verified again if healing step is run.
requirements: ['yaml']

EXAMPLES = '''
# Run all the checks specified in a test suite.
- name: Run all the checks in verify-app-cluster.yml
  standup:
    checks_file_path: verify-app-cluster.yml
    
# Run only checks related to db and web from verify-app-cluster.yml
- name: Run all the checks in verify-app-cluster.yml
  standup:
    checks_file_path: verify-app-cluster.yml
    roles: db,web
    
# Run only checks related to db and web from verify-app-cluster.yml and take corrective action if needed.
- name: Run all the checks in verify-app-cluster.yml and heal state if needed
  standup:
    checks_file_path: verify-app-cluster.yml
    roles: db,web
    heal_state: true
```
# Checks File

The checks that are executed and evaluated by standup module are specified in a YAML file. Multiple checks can be present in a checks file and each check can be tagged with one or more role name so the roles in a file can be executed selectively by indicating the roles to be covered as one of the module parameters.

A sample checks file below:
```
#CentOS smoke checks for standup module.
---
title: Test suite for verifying standup module on CentOS.
checks:
   -  name: "Test common 1"
      description: Check if the OS is CentOS
      command: cat /etc/os-release |grep ^NAME|grep CentOS

   -  name: "Test web 1"
      description: Check if Apache service is running
      roles: web
      command: sudo service httpd status | grep -v "not running"
      heal: sudo service httpd start

   -  name: "Test web 2"
      description: Check if there is any Apache activity
      roles: web
      command: ls /var/log/httpd/access_log 

   -  name: "Test db 1"
      description: Check if mysql service is running
      roles: db
      command: ps -ef |grep mariadb|wc -l
      heal: sudo service mariadb start
      output_compare:
         type: number
         value: 3
         operator: EQ

   -  name: "Test web 3"
      description: Verify if there are more than 2 Apache processes running
      command: ps -ef | grep httpd | grep -v color | wc -l
      output_compare:
         type: number
         value: 2
         operator: GT
```
A check has the following attributes and options:

- name - A label for the check. Required.
- description - A short description for the check. Required.
- roles - One or more tags to indicate the role of infrastructure, delimited by comma. Optional.
- command - The command to run on the target host and the check status is determined based on its result, both OS system status ($? == 0) and optionally the output. Required.
- ignore_status - The system result executing check command is ignored and will be considered success. Default: false.
- heal - If heal action is specified as one of the module options (heal_state=true), this command will be run to heal a state if the check command fails. If heal command succeeds, the check command runs again. Optional.
- output_compare - Output of check command is evaluated as below, optionally:
   - type - str or number, indicating string comparison or numeric comparison of the output.
   - value - The reference value with which the output should be compared.
   - operator - One of the following: GE, EQ, GT, LE and LT. Only EQ is supported for "str" type, all operators are supported for "number" type.

An Ansible task that uses standup module is marked successful if all the checks executed from the input checks file are run successfully.

# Getting Started for Development

Generally follow the steps available at http://docs.ansible.com/ansible/latest/dev_guide/developing_modules_general.html for development and basic testing.

# Testing a New Version 

After the basic tests are passed in the Python development environment, the module must be smoked-tested against Ubuntu and CentOS targets. 

- Clone this repo to your local environment where Ansible is installed.
- Execute the test playbook against Ubuntu and CentOS targets as in the following example:
```
$ ansible-playbook standup-test.yml -i host-centos,host-ubuntu, [-u REMOTE_USER -k] [-K]
```
The actual commandline would depend on how your SSH access is configured on the remote system, and, whether or not the REMOTE_USER has to provide sudo password.
