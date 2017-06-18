### NFS server and client inside kubernetes
**Note:** nfs-server pod doesn't work inside minikube.  
There are two dirs containing manifests to deploy NFS server and client (example nginx deployment).  
Assuming that:  
- node which runs nfs-server has IP `192.168.100.202` and named as `node1` inside cluster.  
  (if required change IP in two files: `nfs-client-example/nfs-client-pv.yaml`, `nfs-server/nfs-server-service.yaml`)  
- client subnet `192.168.0.0/16` is secure and nfs server accepts any connection from it  
  (subnet can be changed in `nfs-server/nfs-server-configmap.yaml`, as other nfs-server parameters)  
- image used for nfs-server: https://github.com/dabealu/docker/tree/master/nfs-ganesha  
  
##### Deploy server
```
# label node to bind nfs-server to it
kubectl label node node1 nfs-server=true

# deploy server
kubectl apply -Rf server
```
  
##### Deploy client
```
# at nfs-server node create file inside exported dir
echo 'mounted index.html' > /var/lib/nfs-export/index.html

# deploy client
kubectl apply -Rf client

# check that volume mounted inside pod, we should see 'mounted index.html'
kubectl exec -ti nfs-client-[press TAB twice] -- cat /usr/share/nginx/html/index.html
```

##### Cleanup
```
# remove client and server objects
kubectl delete -Rf client
kubectl delete -Rf server

# remove node label
kubectl label node node1 nfs-server-

# on nfs-server node, remove exported dir
rm -rf /var/lib/nfs-export
```
