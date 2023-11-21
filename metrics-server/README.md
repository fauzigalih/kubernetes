# Install Metrics Server Kubernetes

### Requirements
- Kubernetes Installed
- Kubernetes node and network is Ready

note: In my case, i've kubernetes with kubeadm.

### Installation 
To install the latest Metrics Server release from the components.yaml manifest, run the following command:
```
$ kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Restart `kubelet` service:
```
$ sudo systemctl restart kubelet
```
```
$ sudo systemctl enable kubelet
```

Check status `metrics-server`:
```
$ kubectl get pod -A
```
![image](https://github.com/fauzigalih/kubernetes/assets/64176403/70596930-5c42-41d2-86e1-95e6d0e1f8f1)

If `metrics-server` is Running and Ready, check metrics with command:
```
$ kubectl top nodes
```
![image](https://github.com/fauzigalih/kubernetes/assets/64176403/950b4868-fb76-462e-8a64-aba987959c83)

### Optional
If check metrics, and have error like this, you must edit and add comman below:<br>
![image](https://github.com/fauzigalih/kubernetes/assets/64176403/4b448761-89f1-44f4-aee3-5f22fae78ac3)

Edit `metrics-server` :
```
$ kubectl -n kube-system edit deployments.apps metrics-server
```

And add this script, adjust position like file `component.yml` or capture below:
```
command:
  - /metrics-server
  - --kubelet-insecure-tls
  - --kubelet-preferred-address-types=InternalIP
```
![image](https://github.com/fauzigalih/kubernetes/assets/64176403/c710a17f-ccb4-4386-a9b8-c0138d66fd2f)

And restart `kubelet` service:
```
$ sudo systemctl restart kubelet
```
```
$ sudo systemctl enable kubelet
```

And check metrics again:
```
$ kubectl top nodes
```
![image](https://github.com/fauzigalih/kubernetes/assets/64176403/950b4868-fb76-462e-8a64-aba987959c83)


### Finish..
