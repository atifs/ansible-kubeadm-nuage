# Ansible playbook and roles to install kubernetes with nuage (version 1.5)

## Goal

Provide an Ansible playbook that implements the steps described in [Installing Kubernetes on Linux with kubeadm](http://kubernetes.io/docs/getting-started-guides/kubeadm/) for version 1.5 (check at the bottom of the kubeadm tutorial)
My case, ansible runs into a <a href="https://pinrojas.com/2017/04/04/data-only-containers-for-ansible-automation/">container with a temporary private key</a>.

## Please Note

Kubernetes/kubeadm 1.5

## Assumptions

These playbooks assume:

* Nuage VSD and VSCs are up and running.
* You have access to 3+ Linux machines, with CentOS 7
* Runs into a <a href="https://pinrojas.com/2017/04/04/data-only-containers-for-ansible-automation/">container with a temporary ssh private key</a>.
* Full network connectivty exists between the machines, and the Ansible control machine (e.g. your computer)
* The machines have access to the Internet
* You are Ansible-knowledgable, can ssh into all the machines, and can sudo with no password prompt
* You have access to your Nuage VSD and VSCs from any server with no restrictions
* Make sure your machines are time-synchronized, and that you understand their firewall configuration and status

## Ansible Inventory Structure and Requirements

1. Define an Ansible inventory group for each Kubernetes cluster you wish to create/manage, e.g. ```k8s_test```
2. Define two Ansible child groups under each cluster group, with names ```_master``` and ```_node```, for example ```k8s_test_master``` and ```k8s_test_node```
3. List the FQDN of the machine assigned the role of Kubernetes master in the ```[cluster_name_master]``` group.
3. List the FQDNs of the machines assigned the role of Kubernetes node in the ```[cluster_name_node]``` group.
5. Optionally add the variable ```master_ip_address_configured``` in the ```[cluster_name_master:vars]``` section, if your master machine has multiple interfaces, and the default interface is NOT the interface you want the nodes to use to connect to the master.

A sample Inventory file called k8s-hosts is included, but if you have/use an existing Ansible inventory, it is a lot easier to just add the structure described above to your existing inventory.

After you have done this, you should be able to succesfully execute something like this:

```
    ansible nuage_k8s -i k8s-hosts -m ping -e cluter_name=nuage_k8s
```

And your master and node machines should respond.  




Don't bother proceeding until all the above work!

## Run the playbook

When you are ready to proceed, run something like:

```
      ansible-playbook -i k8s-hosts -e cluster_name=nuage_k8s nuage-cluster-1.5-create.yml
```

This should execute/implement all four installation steps in the aforementioned installation guide.

If this ends succesfully, then you can proceed to install Nuage VRS and K8s plugins:

```
      ansible-playbook -i k8s-hosts -e cluster_name=nuage_k8s nuage-4.0-k8s-plugin.yml
```

You can try some of the examples at: https://github.com/p1nrojas/nuage-k8s-plugin-4.0r8/tree/master/sample-jsons

If you want to interact with your cluster via the kubectl command on your own machine (and why wouldn't you?), take note of the last note in the "Limitations" section of the guide:

> There is not yet an easy way to generate a kubeconfig file which can be used to authenticate to the cluster remotely with kubectl on, 
> for example, your workstation. Workaround: copy the kubelet’s kubeconfig from the master: use 

>   `scp root@<master>:/etc/kubernetes/admin.conf . `

> and then e.g. 

>   `kubectl --kubeconfig ./admin.conf get nodes` 

> from your workstation.


The playbook retrieves the admin.conf file, and stores it locally as ```./cfg/cluster_name/admin.conf``` to facilitate remote kubectl access.

## Additional Playbooks

```
    ansible-playbook -i k8s-hosts -e cluster_name=nuage_k8s nuage-cluster-1.5-destroy.yml
```

Completely destroys your cluster, with no backups. Don't run this unless that is what you really want!

## Notes and Caveats

* This playbook is under active development, but is not extensively tested.
* I have successfully run this to completion on a 3 machine CentOS setup, it basically worked the first time.

## Acknowlegements

* Kudos to Don Jackson, where actually I forked this project

## Contributing

Pull requests and issues are welcome.












