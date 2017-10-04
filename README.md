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
'''

EXAMPLES = '''
# Run all the checks specified in a test suite.
- name: Run all the checks in verify-app-cluster.yml
  standup:
    checks_file_path: verify-app-cluster.yml
    
# Run only checks related to mysql and web from verify-app-cluster.yml
- name: Run all the checks in verify-app-cluster.yml
  standup:
    checks_file_path: verify-app-cluster.yml
    roles: mysql,web
    
# Run only checks related to mysql and web from verify-app-cluster.yml and take corrective action if needed.
- name: Run all the checks in verify-app-cluster.yml and heal state if needed
  standup:
    checks_file_path: verify-app-cluster.yml
    roles: mysql,web
    heal_state: true
```
# Getting Started for Development

# Testing a New Version 
