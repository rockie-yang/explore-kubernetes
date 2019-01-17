# [Multi Node local kubernetes environment](https://medium.com/devopslinks/deploying-multi-node-kubernetes-environment-in-your-local-machine-a66a1eb82e36)

[get the latest version or some version by check here](https://github.com/kubernetes-sigs/kubeadm-dind-cluster/tree/master/fixed). e.g.

	wget https://cdn.rawgit.com/Mirantis/kubeadm-dind-cluster/master/fixed/dind-cluster-v1.11.sh

on Mac, there might be no file name sha1sum, just create a symbolic link

	sudo ln -s /usr/bin/shasum /usr/local/bin/sha1sum

Start it

	chmod +x dind-cluster-v1.11.sh
	./dind-cluster-v1.11.sh

After a while, 3 nodes cluster will be started. we shall be able to acess [the dashboard](http://127.0.0.1:32768/api/v1/namespaces/kube-system/services/kubernetes-dashboard:/proxy)


Check process on master, it has 5 pairs of process. kube-proxy, controller-manager, scheduler, etcd, api-server

	docker exec kube-master bash

	root@kube-master:/#  docker ps --format '{{.Names}}'
	k8s_kube-proxy_kube-proxy-nqbjv_kube-system_56c8d948-db36-11e8-9c44-2227d0d49b64_1
	k8s_POD_kube-proxy-nqbjv_kube-system_56c8d948-db36-11e8-9c44-2227d0d49b64_1

	k8s_kube-controller-manager_kube-controller-manager-kube-master_default_f0d02eb1f73edc0d1b1cdf90720fdeb4_1
	k8s_POD_kube-controller-manager-kube-master_default_f0d02eb1f73edc0d1b1cdf90720fdeb4_1

	k8s_kube-scheduler_kube-scheduler-kube-master_default_27a9e6670b45047dfa1152fa922aff56_1
	k8s_POD_kube-scheduler-kube-master_default_27a9e6670b45047dfa1152fa922aff56_1

	k8s_etcd_etcd-kube-master_kube-system_78263d83ff9d8e4fa24f4ff1b321f5b4_1
	k8s_POD_etcd-kube-master_kube-system_78263d83ff9d8e4fa24f4ff1b321f5b4_1

	k8s_kube-apiserver_kube-apiserver-kube-master_default_ec74441bf1f808a17b5ac727e17c5e67_1
	k8s_POD_kube-apiserver-kube-master_default_ec74441bf1f808a17b5ac727e17c5e67_1

and kubelet which is not running as docker

	systemctl status kubelet

	ps -ef | grep kubelet

Using ps -ef --forest to see hierachy

Check processes on node. It has kube-dns, kube-proxy. 
sidecar and dnsmasq shall be for dns. 

	docker exec -it kube-node-1 bash

	k8s_sidecar_kube-dns-57f756cc64-fgnmt_kube-system_84774570-db36-11e8-a505-2227d0d49b64_0
	k8s_dnsmasq_kube-dns-57f756cc64-fgnmt_kube-system_84774570-db36-11e8-a505-2227d0d49b64_0

	k8s_kubedns_kube-dns-57f756cc64-fgnmt_kube-system_84774570-db36-11e8-a505-2227d0d49b64_0
	k8s_POD_kube-dns-57f756cc64-fgnmt_kube-system_84774570-db36-11e8-a505-2227d0d49b64_0

	k8s_kube-proxy_kube-proxy-dvtx4_kube-system_52486f67-db36-11e8-9c44-2227d0d49b64_1
	k8s_POD_kube-proxy-dvtx4_kube-system_52486f67-db36-11e8-9c44-2227d0d49b64_1

and kubelet which is not running as docker

	systemctl status kubelet

	ps -ef | grep kubelet


