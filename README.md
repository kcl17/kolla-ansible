# Kolla-Ansible Deployment Guide

This guide walks you through deploying OpenStack using Kolla Ansible.

## ✅ Host Machine Requirements

Ensure your host meets the following minimum requirements:

    2 network interfaces

    8 GB RAM

    40 GB disk space

## ✅ Install Dependencies

For Ubuntu/Debian

    sudo apt update
    sudo apt install -y git python3-dev libffi-dev gcc libssl-dev libdbus-glib-1-dev python3-venv

For CentOS or Rocky, run:

    sudo dnf install git python3-devel libffi-devel gcc openssl-devel python3-libselinux


## ✅ Set Up a Python Virtual Environment (Recommended)

> Activate the environment before running any Kolla commands.

    python3 -m venv ~/kolla-venv
    source ~/kolla-venv/bin/activate
    pip install -U pip


## ✅ Install Kolla Ansible

    git clone --branch master https://opendev.org/openstack/kolla-ansible
    cd kolla-ansible
    pip install ./kolla-ansible

## ✅ Configure Kolla

### Create Configuration Directory

    sudo mkdir -p /etc/kolla
    sudo chown $USER:$USER /etc/kolla
    
### Copy Configuration Files
Copy the configuration files to /etc/kolla directory. kolla-ansible holds the configuration files (globals.yml and passwords.yml) in etc/kolla.

    cp -r kolla-ansible/etc/kolla/* /etc/kolla
    cp kolla-ansible/ansible/inventory/* .

## ✅ Install Ansible Galaxy Requirements

    kolla-ansible install-deps


## ✅ Prepare Initial Configuration
### Inventory File

Kolla provides:

  . `all-in-one`: For single-node deployment.

  . `multinode`: For multi-node environments.

This guide uses the **all-in-one** inventory.

### Generate Passwords

    cd kolla-ansible/tools
    ./generate_passwords.py

### Edit /etc/kolla/globals.yml

Basic required fields:

    kolla_base_distro: "rocky"         # or "ubuntu"
    openstack_release: "2024.1"        # or your desired version
    network_interface: "eth0"
    neutron_external_interface: "eth1"
    kolla_internal_vip_address: "10.1.0.250"

For AArch64 systems, add:

    openstack_tag_suffix: "-aarch64"

### Optional: Enable Additional Services

You can enable services by adding flags like:

    enable_cinder: "yes"
    enable_heat: "yes"

To keep your config modular, use:

    mkdir -p /etc/kolla/globals.d
    # Place additional .yml files inside this directory


Example: /etc/kolla/globals.d/cinder.yml

## ✅ Deploy OpenStack

### Bootstrap Hosts

    cd kolla-ansible/tools
    ./kolla-ansible bootstrap-servers -i ../../all-in-one

### Run Prechecks

    kolla-ansible prechecks -i ../../all-in-one

### Deploy

    kolla-ansible deploy -i ../../all-in-one

## ✅ Post-Deployment

### Setup Admin OpenRC

    ./kolla-ansible post-deploy

This generates clouds.yaml in /etc/kolla/. Copy it to:

    mkdir -p ~/.config/openstack
    cp /etc/kolla/clouds.yaml ~/.config/openstack/

Or set the env variable:

    export OS_CLIENT_CONFIG_FILE=/etc/kolla/clouds.yaml

## ✅ Use OpenStack

### Install the CLI

    pip install python-openstackclient -c https://releases.openstack.org/constraints/upper/master

### Run Example Initialization Script

    cd kolla-ansible/tools
    ./init-runonce

This sets up demo networks, images, flavors, etc.

## 📚 Resources

- [Kolla Ansible Docs](https://docs.openstack.org/kolla-ansible/latest/)
- [Network Overview](https://docs.openstack.org/kolla-ansible/latest/user/networking.html)
- [Service Configuration Guide](https://docs.openstack.org/kolla-ansible/latest/reference/services.html)
