# Day 3

## Lab - Creating a ClusterIP Internal service for nginx deployment

Let's first delete the nginx node-port service
```
oc get svc
oc delete svc/nginx
```

Let's create a nginx deployment with 3 pods if you don't have it already
```
oc project jegan
oc create deploy nginx --image=bitnami/nginx:latest --replicas=3
oc expose deploy/nginx --type=ClusterIP --port=8080
oc get svc
oc describe svc/nginx
oc get po -o wide
```

In order to access the clusterip internal service, let's create a test pod
```
oc run test-pod --image=tektutor/spring-ms:1.0
oc get po
oc rsh pod/test-pod sh
curl http://nginx:8080
curl http://<service-name>:<service-port>
curl http://10.217.5.158:8080
curl http://<service-ip>:<service-port>
```

Expected output
![image](https://github.com/user-attachments/assets/3020315d-0af3-45e1-9c04-cd33ecf7e6b1)
![image](https://github.com/user-attachments/assets/f252d0ce-650f-480e-89bb-28eff57ddb7d)
![image](https://github.com/user-attachments/assets/8c552613-2446-49cc-9c9f-0137b39656c9)
![image](https://github.com/user-attachments/assets/ac6d19ee-2db3-45ec-a623-7120ec76e40e)
![image](https://github.com/user-attachments/assets/f971fb95-3acd-4167-a1b5-9bdbf164d612)
![image](https://github.com/user-attachments/assets/8f414317-ac49-4464-8559-798fcea32f4f)

## Info - Metallb Operator
<pre>
https://medium.com/tektutor/using-metallb-loadbalancer-with-bare-metal-openshift-onprem-4230944bfa35  
</pre>

## Lab - Creating an external route with public url
```
oc get deploy
oc get svc
oc delete svc/nginx
oc expose deploy/nginx --port=8080
oc get svc
oc describe svc/nginx
oc expose svc/nginx
curl http://nginx-jegan.apps-crc.testing
```

Expected output
![image](https://github.com/user-attachments/assets/e0f50e33-4965-48dd-955b-9ecab0375161)
![image](https://github.com/user-attachments/assets/cdf404cb-abf0-4955-881c-cce9c754cf2e)
![image](https://github.com/user-attachments/assets/2172b082-5560-4a7a-a18b-415ad2bc5262)

## Lab - Getting inside a openshift node 
```
oc debug node/crc
chroot /host
crictl ps
podman images
exit
```

## Lab - Creating a loadbalancer service for nginx deployment
```
oc delete svc/nginx
oc expose deploy --type=LoadBalancer --port=8080
oc get svc
oc describe svc/nginx
curl http://<loadbalancer-external-p>
```

## Info - Openshift Operators
<pre>
- automating the human operator specific skills within Openshift cluster
- Operators = Custom Resources(s) + Custom Controller(s)
- One Controller manages one type of Resource
- Operators extend the functionanlity of Openshift/Kubernetes
</pre>

## Info - How Custom Resource(CR) are added to Openshift Cluster?
<pre>
- We need to define Custom Resource Definition(CRD)
- CRDS introduce/register a new type of Custom Resource(CR) to your Openshift Cluster
</pre>

## Lab - Deploying nginx web server in declarative style using yaml files
```
oc project jegan
oc new-project jegan
oc create deploy nginx --image=bitnami/nginx:latest --replicas=3 --dry-run=client -o yaml
oc create deploy nginx --image=bitnami/nginx:latest --replicas=3 --dry-run=client -o yaml > nginx-deploy.yml
oc create -f nginx-deploy.yml --save-config
oc get deploy,rs,po
```

Expected output
![image](https://github.com/user-attachments/assets/43ecb9e5-70cb-4139-9f0e-867f8ab13344)
![image](https://github.com/user-attachments/assets/ba86e670-3213-4f28-bacd-513eb112fcca)
![image](https://github.com/user-attachments/assets/8877703a-83c9-49dc-bb8d-746f8117fdba)

## Lab - Updating the deployment in declarative style - scale up/down
Update the replicas count from 3 to 5 and save it
```
ls
vim nginx-deploy.yml
oc apply -f nginx-deploy
oc get po
```

Expected output
![image](https://github.com/user-attachments/assets/574869eb-f31a-4edd-9133-4cc554dac466)
![image](https://github.com/user-attachments/assets/d30b7b67-9c13-457c-b092-ebd0affe8ff8)
![image](https://github.com/user-attachments/assets/c444d1d8-1348-49d0-9d9b-a0a8331ccbff)

## Lab - Delete a deployment in declarative style
```
oc get deploy,rs,po
oc delete -f nginx-deploy.yml
oc get deploy,rs,po
```

Expected output
![image](https://github.com/user-attachments/assets/7eea1272-b022-444b-99f3-9af679a9107e)

## Lab - Declaratively creating replicaset without deployment
```
oc create -f nginx-deploy.yml
oc get rs
oc get rs/nginx-66c775969 -o yaml
oc get rs/nginx-66c775969 -o yaml > nginx-rs.yml
# We need to clean up the nginx-rs.yml as shown below
cat nginx-rs.yml
oc delete -f nginx-deploy.yml
oc get deploy,rs,po
oc create -f nginx-rs.yml --save-config
oc get rs,po
```

Expected output
![image](https://github.com/user-attachments/assets/29e1aa72-8570-443c-89ba-8f9c9929c4ba)
![image](https://github.com/user-attachments/assets/e705b394-8f89-4cf4-82e2-44106e5fe8fc)
![image](https://github.com/user-attachments/assets/4ca6bc6d-a119-4cfb-b869-d6cdd3feabf0)
![image](https://github.com/user-attachments/assets/efe9335b-0fed-44b9-8baa-d817d9c279f6)
![image](https://github.com/user-attachments/assets/4625c4f0-dcba-442d-838e-abe793fadf61)
![image](https://github.com/user-attachments/assets/f99a15a8-9a74-4cf5-83d0-df2ba6d66a52)
![image](https://github.com/user-attachments/assets/2dfc8ee3-b3ea-40c5-bf68-f89b260dbca9)

## Lab - Creating a pod in declarative style without deployment and replicaset
```
oc get po
oc get po/nginx-rs-4tqsj -o yaml
oc get po/nginx-rs-4tqsj -o yaml > nginx-pod.yml
# Clean up the pod as shown below
cat nginx-pod.yml
oc delete -f nginx-rs.yml
oc create -f nginx-pod.yml --save-config
oc get po
```

Expected output
![image](https://github.com/user-attachments/assets/75db5209-7ad2-49c2-ad26-e4451db018b9)
![image](https://github.com/user-attachments/assets/318c136c-f86c-4cd5-8662-fe3945934a4b)
![image](https://github.com/user-attachments/assets/38917c0c-7dae-4cc5-a932-53a0cafe3cf0)
![image](https://github.com/user-attachments/assets/ee0259ce-4fab-4350-9a45-3700d56633c7)
![image](https://github.com/user-attachments/assets/ea78b773-317f-4907-8df2-cc1c0d5fb9ce)
![image](https://github.com/user-attachments/assets/3d804d90-aae1-4f4c-aebb-c6c32750b106)

Once you are done with this exercise, you may delete the pod declaratively as shown below
```
oc get po
oc delete -f nginx-pod.yml
oc get po
```

Expected output

![image](https://github.com/user-attachments/assets/9de76267-45c6-4ca3-9270-8c8dcf2acf0b)

## Lab - Declaratively creating an internal clusterip service for nginx deployment
```
oc create -f nginx-deploy.yml
oc expose deploy/nginx --type=ClusterIP --port=8080 --dry-run=client -o yaml
oc expose deploy/nginx --type=ClusterIP --port=8080 --dry-run=client -o yaml > nginx-clusterip-svc.yml
```

Expected output

## Lab - Declaratively creating an internal loadbalancer service for nginx deployment
```
oc create -f nginx-deploy.yml
oc expose deploy/nginx --type=LoadBalancer --port=8080 --dry-run=client -o yaml
oc expose deploy/nginx --type=LoadBalancer --port=8080 --dry-run=client -o yaml > nginx-lb-svc.yml
```

Expected output

## Lab - Declaratively creating an internal nodeport service for nginx deployment
```
oc create -f nginx-deploy.yml
oc expose deploy/nginx --type=NodePort --port=8080 --dry-run=client -o yaml
oc expose deploy/nginx --type=NodePort --port=8080 --dry-run=client -o yaml > nginx-nodeport-svc.yml
```

Expected output


## Info - Persistent Volume (PV)
<pre>
- is a external storage that can be used by the applications runing within Pod
- this can be provisioned by Administrator either manually or dynamically 
- created on the cluster scope, which any pod running in any project namespace can claim and use PV
- In case the PV is manually provisioned, the administrator will have create Persistent volume 
  - with a specific size capacity
  - with specific access modes
    - ReadWriteOnce
    - ReadWriteMany
    - etc
  - StorageClass(optional)
  - Labels ( optional )
</pre>

## Info - Persistent Volume Claims (PVC)
<pre>
- is the way your application can request for external storage
- PV will have to define
  - the size of the storage required
  - storageclass(optional)
  - access mode
  - labels (optional)
</pre>

