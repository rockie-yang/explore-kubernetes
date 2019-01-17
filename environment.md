
# Setup Environment - this method is DEPRECATED by kubernetes

## Install Vagrant

Vagrant is a tool for building complete development environments. 
I always having vagrant installed on my computer.

Go to https://www.vagrantup.com/downloads.html. 
Find a suitable package download and install.

## Install Virtualbox

Virtualbox is a tool for create virtual machines.
The good thing about virtualbox is I can installed on what ever OS I have.

Go to https://www.virtualbox.org/wiki/Downloads. 
Find a suitable package download and install.

## Install GIT

Go to https://git-scm.com/downloads.
Find a suitable package download and install.

## Setup Lab E

Clone kubernetes repo.
 
    https://github.com/kubernetes/kubernetes.git

Create a file `kubernetes/cluster/env.sh` with content like this.

    #!/bin/bash

    # With this like the hostname will be master, node-1, etc
    # otherwise it will be kubernetes-master, kubernetes-node-1, etc
    export node_prefix=""
    
    # when want to use vagrant
    export KUBERNETES_PROVIDER=vagrant
    
    # The default setting works well on machine with 8G memory
    # Default memory for master is 1280M, for node is 1024
    # When I have 16G memory, I increase it a bit
    #export KUBERNETES_MASTER_MEMORY=2048
    #export KUBERNETES_NODE_MEMORY=2048
    
    # when want to use virtualbox
    export VAGRANT_DEFAULT_PROVIDER=virtualbox
    
    # 3 nodes is perfect for experience full cluster feature
    export NUM_NODES=3
    
Init Environment. The kube-up need quite some minute to run.

    cd kubernetes
    cluster/kube-up.sh

You will run following command if kube-up run failed for some reason.
    
    cluster/kube-down.sh
    cluster/kube-up.sh
    
Save the environment

    vagrant snapshot save init
    
Later on if there are some thing strange happen. 
Or the machine get restarted.
Run the following command to get the environment back

    vagrant snapshot restore init

