Setting up Kubernetes can be a procedure that takes some time, but with the Kubernetes distribution [K3S](https://k3s.io) and a Ansible playbook we can get a Kubernetes cluster up and running within 5min.

So I created just that, an Ansible playbook where you just add your nodes to a inventory file and then the playbook will install all the dependencies we need for K3S, the workers will automaticly join the master AND I have also added some firewall rules (with help from [UFW](https://help.ubuntu.com/community/UFW)) so you have some basic protection for your servers.

## Requirements
* Git
* Virtualenv ([Python](https://virtualenv.pypa.io/en/latest/)) on the computer you will deploy this from.
* At least 2 servers with SSH access.
  * Must be based on Ubuntu.
  * User with sudo permissions.
  * Must have docker installed.

## Preparations

* Point a hostname to the server you want to be your `kubernetes_master_server`.

## Steps

### Step 1

Prepare your local environment (or wherever you deploy this from) with the dependencies we need. This will install a Virtual Python environment and download Python, Pip and [Ansible](https://www.ansible.com) and make those available in your `$PATH` so you can execute them.

1. Clone the K3S Ansible repository from [Github](https://github.com/pixelpiloten/k3s-ansible).

    ```bash
    $ git clone git@github.com:pixelpiloten/k3s-ansible.git
    ```

2. Go to the `ansible` directory and create your Virtual Python environment.

    ```bash
    $ virtualenv venv
    ```

3. Activate the Virtual Python environment.

    ```bash
    $ source venv/bin/activate
    ```

### Step 2

Configure your inventory file with the servers you have, you can add how many workers you want but there can only be one master in K3S (though it might be supported in the future).

1. Rename the `inventory.example` file to `inventory`.

    ```bash
    $ mv inventory.example inventory
    ```

2. Change the `kubernetes_master_server` to the server you want to be your Kubernetes master and add ip(s) to the `allowed_kubernetes_access` list for the ips that should have access to the cluster via `Kubectl`. Alsop change `ansible_ssh_host`, `ansible_ssh_user`, `ansible_ssh_port`, `ansible_ssh_private_key_file` and `hostname_alias` to your server login details. And of course add how many workers you have.

    ```yaml
    all:
    vars:
      ansible_python_interpreter: /usr/bin/python3
      k3s_version: v0.9.1 # Corresponds with the release on github.
      kubernetes_master_server: https://node01.example.com:6443 # Change to your master server
      allowed_kubernetes_access: # Change these to a list of ips outside your cluster that should have access to the api server.
        - 1.2.3.4
        - 1.2.3.5
    children:
      k3scluster:
        hosts:
          node01: # We can only have one master.
            ansible_ssh_host: node01.example.com
            ansible_ssh_user: ubuntu
            ansible_ssh_port: 22
            ansible_ssh_private_key_file: ~/.ssh/id_rsa
            hostname_alias: node01
            kubernetes_role: master # Needs to be as it is.
            node_ip: 10.0.0.1
          node02: # Copy this and add how many workers you want and have.
            ansible_ssh_host: node02.example.com
            ansible_ssh_user: ubuntu
            ansible_ssh_port: 22
            ansible_ssh_private_key_file: ~/.ssh/id_rsa
            hostname_alias: node02
            kubernetes_role: worker # Needs to be as it is.
            node_ip: 10.0.0.2
    ```

### Step 3

You are now ready to install K3S on your servers. The Ansible playbook will go through the inventory-file and install the dependencies on your servers and then install K3S on your master and your workers, the workers will automaticly join your K3S master. At the end the playbook will write a Kubeconfig-file to `/tmp/kubeconfig.yaml` on the machine you are running the playbook on.

1. Run the Ansible playbook and supply the sudo password when Ansible asks for it.

    ```bash
    $ ansible-playbook -i inventory playbook.yml --ask-become-pass
    ```

2. When the playbook is done you can copy the Kubeconfig-file `/tmp/kubeconfig.yaml` to your `~/.kube/config` or where ever you want to keep it, BUT you need to modify the the `server` hostname to whatever your `kubernetes_master_server` is. (PS. Do not use the content below, use the `/tmp/kubeconfig.yaml` that got generated locally)

    ```yaml
    apiVersion: v1
    clusters:
    - cluster:
        certificate-authority-data: <REDACTED-CERTIFICATE>
        server: https://0.0.0.0:6443
    name: default
    contexts:
    - context:
        cluster: default
        user: default
    name: default
    current-context: default
    kind: Config
    preferences: {}
    users:
    - name: default
    user:
        password: <REDACTED-PASSWORD>
        username: admin
    ```

### Step 4

Start deploying applications to your new K3S cluster :)