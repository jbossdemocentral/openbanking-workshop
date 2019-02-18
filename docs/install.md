# Open Banking Workshop Installation

This page describes the installation of the Open Banking Workshop from the latest sources from GitHub.

## Pre-requisites

You will need an OpenShift Container Platform to install this workshop on. You can order a vanilla provisioning from the Red Hat Product Demo System (RHPDS) following this [instructions](https://mojo.redhat.com/docs/DOC-1175640).

To install the Open Banking Workshop, you need to have a host machine with the latest stable release version of the OpenShift client tools. RHPDS provides a bastion machine to do this.

You'll want to know how to [fork](https://help.github.com/articles/fork-a-repo/) and [clone](https://help.github.com/articles/cloning-a-repository/) a Git repository, and how to [check out a branch](https://git-scm.com/docs/git-checkout#git-checkout-emgitcheckoutemltbranchgt).

The Open Banking Workshop can be installed using automated Ansible playbooks or following manual steps.

First you will need to install integr8ly for having the components available and then the actual workshop to configure the multitenancy and attendants security realms.

## Installing using Ansible

We provide an Ansible playbook to install all the required components and software for this workshop.

Installing with Ansible requires creating an inventory file with the variables for configuring the system. Example inventory files can be found in the ansible/inventory folder.

### Procedure to install Integr8ly

The recommended way to install the workshop is running the ansible playbook from the OpenShift cluster bastion machine. This is the fastest way to run the installer as it's already running in the cluster closest to the master node.

1. Login to the bastion machine following the email instructions.

    ```bash
    ssh -i /path/to/ocp_workshop.pem ec2-user@bastion.GUID.openshiftworkshop.com
    ```

    Remember to update the *GUID* with your cluster environment variable and the path to the downloaded PEM file.

1. Git Clone the Integr8ly installation repository.

    ```bash
    git clone -b release-1.1.0 https://github.com/integr8ly/installation.git
    ```

1. Change folder to installation base.

    ```Bash
    cd installation/evals/
    ```
    
1. Edit the default inventory according to your environment *GUID*.

    ```Bash
    vi inventories/hosts.template
    ```
    
1. Replace `127.0.0.1` under `[master]` with `master1.GUID.internal` where *GUID* is your environment identifier.

    ```yaml
    [local:vars]
    ansible_connection=local
    
    [local]
    127.0.0.1
    
    [OSEv3:children]
    master
    
    [OSEv3:vars]
    ansible_user=ec2-user
    
    [master]
    master1.GUID.internal
    ```

1. Become super user running the following command.

    ```bash
    sudo su
    ```

1. Run the Ansible playbook. Replace `eval_seed_users_count=101` with the actual number of users you want to create for the workshop.

    ```bash
    ansible-playbook -i inventories/hosts.template playbooks/install.yml -e eval_seed_users_count=101 -e rhsso_seed_users_name_format=user%d -e rhsso_seed_users_password=openshift
    ```

It will a take a couple of minutes to install the integr8ly environment. After the install is complete you can then proceed to install the actual workshop.

### Procedure to install Workshop

After you install integr8ly, is now time to install and configure the workshop environment.

1. If you are still in the same session exit the root user.

    ```Bash
    exit
    ```

1. Get back to the user home.

    ```bash
    cd
    ```
    
1. Clone the Open Banking Workshop.

    ```bash
    git clone https://github.com/jbossdemocentral/openbanking-workshop.git
    ```
    
1. Change folder to the support install directory.

    ```bash
    cd openbanking-workshop/support/install/ansible/
    ```
    
1. Edit the inventory with the correct environment hostnames.

    ```bash
    vi inventory/integreatly.example
    ```

1. Replace `master.akeating.openshiftworkshop.com` under `[master]` with `master1.GUID.internal` where *GUID* is your environment identifier. Replace the `ocp_domain` and the `ocp_apps_domain` with your environment *GUID*. 

    ```yaml
    ...
    
    [master]
    master1.GUID.internal
    ...
    
    [workshop:vars]
    sso_project=sso
    gogs_project=gogs
    microcks_project=microcks
    apicurio_project=apicurio
    namespace=threescale
    backend_project=international
    configure_only=false
    ocp_domain=GUID.openshiftworkshop.com
ocp_apps_domain=apps.GUID.openshiftworkshop.com
    usersno=101
    che=false
    
    ...

    ```

1. Become super user running the following command.

    ```bash
    sudo su
    ```

1. Run the Ansible playbook.

    ```bash
    ansible-playbook -i inventory/integreatly.example playbooks/openshift/integreatly-install.yml
    ```
