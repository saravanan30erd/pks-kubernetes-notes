# Summary

You are getting Econnection Reset (network errors) in your application running on PKS k8s containers.

# Troubleshoot

## On container

Get the ethernet details
```
cat /sys/class/net/eth0/iflink
421
```

## On Worker Node (where the container is running)

Find the container ethernet on Worker node,
```
ip link |grep ^421:
```

Run the TCPDUMP for that interface,
```
tcpdump -n -tttt -i ef5678erxthji45 port 443 -w conn_reset.pcap
```
Destination port is 443

Copy the file from worker node to some container (its not possible to copy from the worker node directly)
```
docker inspect container --> find the merged directory
cd /var/vcap/store/docker/docker/overlay2/678etryfgchbd3546578hdhghry6373bfnjdhy73848bdnnsh366485ufnmsksuf/merged
cp -rpf /root/conn_reset.pcap .
```
Now the file is under one pod/container.

Now we can use kubectl command to get the file,
```
kubectl cp <namespace>/<pod name>:/conn_reset.pcap conn_reset.pcap
```

## On Wireshark

Use the below filters to find the connect reset packets,
```
Filter reset packets: tcp.flags.reset==1
Filter reset packets with source IP: tcp.flags.reset==1 and ip.src==<SOURCE IP>
Filter only one connection flow: tcp.stream eq 1226
```
