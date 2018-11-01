# Jenkins on OpenShift

**Using Jenkins to Control Application Promotion between OpenShift Clusters**

Reference Documentation:

https://access.redhat.com/documentation/en-us/reference_architectures/2017/html-single/application_cicd_on_openshift_container_platform_with_jenkins/

Forked from:

https://github.com/RHsyseng/jenkins-on-openshift


## Overview

This repo has 3 primary components:

- application: code to deploy the example application
- ansible: configuration for the OpenShift environments and Jenkins pipeline bootstrapping
- jenkins: Jenkins master configuration and declarative Jenkinsfiles

## Layout

```
├── ansible                  # ansible playbooks to configure clusters, create openshift objects
├── app                      # target application being deployed
├── jenkins                  # Jenkins configuration
├── Jenkinsfile              # Main application pipeline
├── Jenkinsfile.release      # Production release pipeline
├── src                      # Jenkins library code
├── Vagrantfile              # Vagrantfile for running RHEL-based clients, oc and ansible-playbook
└── vars                     # Jenkins groovy method for Utils library
```

## Vagrant

The Vagrantfile is provided to bootstrap a local RHEL-based workstation pre-installed with client tools 'oc' and 'ansible-playbook'.

**Requirements**

- 'vagrant-triggers' plugin

        vagrant plugin install vagrant-triggers
- Install Red Hat Enterprise Linux vagrant box. [Download](https://developers.redhat.com/products/rhel/download/)


## Instructions

Docs and configuration updated for v3.11

Custom configured files for this environment:

```
ansible/group_vars/*.yml
configuration/hudson.tasks.Mailer.xml
```

Shortcut instructions from the documents above

```bash
# Clone this repo
git clone https://github.com/eformat/jenkins-on-openshift.git
cd jenkins-on-openshift/ansible

# Create configuration
#for i in group_vars/*.example; do mv "${i}" "${i/.example}"; done

# Ansible setup
ansible-playbook -i inventory.yml main.yml --extra-vars token=$(oc whoami -t)

# Check jenkins custom build completes and it starts up OK
oc login
oc project dev
oc get build -l 'buildconfig=jenkins-custom' --template '{{with index .items 0}}{{.status.phase}}{{end}}'
oc get pod -l 'name==jenkins'
oc logs -f dc/jenkins 

# start app build pipeline
oc start-build app-pipeline

# promote app pipeline
oc start-build release-pipeline

# delete / tidy everything in this lab
oc delete project dev prod stage registry
```
