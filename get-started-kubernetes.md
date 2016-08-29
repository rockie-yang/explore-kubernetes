# Get Started with Kubernetes

## Run nginx
I can run a container similar with docker command

	kubectl run nginx --image=nginx:1.10.0

## Where is nginx running?

Where does the container is running? 
So run the folowing command in kubernetes master. 
There is no containers running on master with name nginx.

	[vagrant@master ~]$ sudo docker ps | grep nginx
	[vagrant@master ~]$ 

It should be running no node-1. So I run the folowing command in kubernetes node-1.

	[vagrant@node-1 ~]$ sudo docker ps | egrep "CONTAINER|nginx"
	
| CONTAINER ID |       IMAGE  |   COMMAND    |    CREATED  | STATUS  | PORTS |    NAMES |
|--------------|--------------|--------------|-------------|---------|-------|----------|
| 4379d9c6402f | nginx:1.10.0 |"nginx -g 'daemon off"| 4 hours ago | Up 4 hours | |k8s_nginx. 41e4c49 _nginx - 1221891139 - e5xxt _ ...|
| d4a87dd6dce7 | gcr.io/google_containers/pause-amd64:3.0 | "/pause" | 4 hours ago | Up 4 hours || k8s_POD. d8dbe16c _nginx - 1221891139 - e5xxt _ ...|

The first container is the one which running real nginx.

	[vagrant@node-1 ~]$ sudo docker exec 4379d9c6402f ps -ef
	UID        PID  PPID  C STIME TTY          TIME CMD
	root         1     0  0 06:27 ?        00:00:00 nginx: master process nginx -g daemon off;
	nginx        9     1  0 06:27 ?        00:00:00 nginx: worker process
	root        10     0  0 07:30 ?        00:00:00 ps -ef
	
The second container, pause, is something different. Some clue on [stackoverflow](http://stackoverflow.com/questions/33472741/what-work-does-the-process-in-container-gcr-io-google-containers-pause0-8-0-d#). The pause source on [github](https://github.com/kubernetes/kubernetes/tree/master/build/pause). A good explaination is TODO

	[vagrant@node-1 ~]$ sudo docker exec d4a87dd6dce7 ps -ef
	exec: "ps": executable file not found in $PATH

## How can I access the nginx inside the cluster?

Check what is the IP for the nginx pod

	[vagrant@master ~]$ kubectl describe pod nginx | grep IP
	IP:		10.246.19.3

I can access from any nodes inside the cluster

	[vagrant@master ~]$ curl 10.246.19.3 2>&1 | grep title
	<title>Welcome to nginx!</title>

	[vagrant@node-1 ~]$ curl 10.246.19.3 2>&1 | grep title
	<title>Welcome to nginx!</title>
	
	[vagrant@node-2 ~]$ curl 10.246.19.3 2>&1 | grep title
	<title>Welcome to nginx!</title>

But I can not access from outside world, the IP 10.246.19.3 is invisible.	
The same as in docker, the container is completely isolated. It needs some mechanism to communicate with outside world. 

## Expose using NodePort

NodePort is one way to expose a service

	[vagrant@master ~]$ kubectl expose deployment nginx --port 80 --type NodePort
	service "nginx" exposed


Get the Port 

	[vagrant@master ~]$ kubectl describe service nginx | grep NodePort:
	NodePort:		<unset>	30584/TCP

It has the communication flow setup like this

![NodePort](k8s-nodeport.png)


It can not access from the master machine.

	[vagrant@master ~]$ curl 127.0.0.1:30584
	curl: (7) Failed to connect to 127.0.0.1 port 30584: Connection refused

While it can access from all other nodes.

	[vagrant@node-1 ~]$ curl 127.0.0.1:30584 2>&1 | grep title
	<title>Welcome to nginx!</title>

	[vagrant@node-2 ~]$ curl 127.0.0.1:30584 2>&1 | grep title
	<title>Welcome to nginx!</title>

I can get node-1's ip address. 

	[vagrant@node-1 ~]$ ifconfig enp0s8 | grep inet
        inet 10.245.1.3  netmask 255.255.255.0  broadcast 10.245.1.255
        inet6 fe80::a00:27ff:feb5:c29a  prefixlen 64  scopeid 0x20<link>

Then access on my host machine.

	rockieyang$ curl 10.245.1.3:30584 2>&1 | grep title
	<title>Welcome to nginx!</title>
	
I can also get node-2's ip address.

	[vagrant@node-2 ~]$ ifconfig enp0s3 | grep inet
        inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255
        inet6 fe80::a00:27ff:fe69:954a  prefixlen 64  scopeid 0x20<link>
        
Then access on my host machine.

	rockieyang$ curl 10.245.1.3:30584 2>&1 | grep title
	<title>Welcome to nginx!</title>

enp0s3 is NAT, so it can only access from the node to external network. And all enp0s3 has the same ip address.

	[vagrant@master ~]$ ifconfig enp0s3 | grep "inet "
        inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255

	[vagrant@node-1 ~]$ ifconfig enp0s3 | grep "inet "
        inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255
                
	[vagrant@node-2 ~]$ ifconfig enp0s3 | grep "inet "
        inet 10.0.2.15  netmask 255.255.255.0  broadcast 10.0.2.255

If we add port forward on the virtual machine, then those service can be accessed by outside world with the host IP.

	[vagrant@node-2 ~]$ ifconfig enp0s8 | grep inet
        inet 10.245.1.4  netmask 255.255.255.0  broadcast 10.245.1.255
        inet6 fe80::a00:27ff:feb2:63a7  prefixlen 64  scopeid 0x20<link>