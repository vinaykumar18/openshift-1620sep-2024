# Day 2

## Kubelet Overview
<pre>
- Kubelet is a Container agent that runs as a service in every node 
- Kubelet is the one that communicates with CRI-O container runtime to mange images and containers
</pre>  


## Pod Overview
<pre>
- Logical grouping of containers
- containers is where our application will be running
</pre>  

## Info - Control Plane Components
<pre>
1. API Server
2. etcd database
3. scheduler
4. controller managers
</pre>  

## API Server Overview
<pre>
- is the heart of Kubernetes/Openshift
- is a Pod that runs in every master node
- API Server has REST APIs for every features supported in Openshift
- API Server stores cluster status and application staus in the etcd database
- API Server is the only component that writes information to the etcd database
- all the Openshift components they only communicate with API Server via REST calls
</pre>

## etcd 
<pre>
- key/value distributed datastore
- is a Pod that runs in every master node
- opensource independent project  
- many etcd instances run as a cluster
- when one instance of etcd database is updated, the other etcd databases in the cluster gets synchronized
</pre>

## Scheduler
<pre>
- Scheduler is a Pod that runs in every master node   
- responsible for scheduling new pods into any one of the health node within openshift cluster
- its gets notifications from API Server when new Pods are created
- scheduler can't schedule pod directly, it will just send its scheduling recommendations to API Server via REST calls
</pre>

## Controller Managers
<pre>
- a group of controllers
- is a collection of many 
- is a Pod that runs in every master node
- controller is an application that runs in a never ending while loop 
- each controller manages one type of Openshift resource
- For example
  - Deployment Controller manages Deployment
  - ReplicaSet Controller manages ReplicaSet
  - Job Controller manages Jobs
  - CronJob Controller manages CronJobs
  - StatefulSet Controller manages StatefulSet
  - DaemonSet Controller manager DaemonSet
  - Endpoint Controller manages Pod endpoints used in services
</pre>

## Info - What happens when we create a new deploy in openshift
```
oc create deployment nginx --image=bitnami/nginx:latest --replicas=3
```

The below chain of activity happens
<pre>
- oc client tool makes a REST call to API server,request to create new deployment with name nginx
- API server receives the request from oc client tool, it then creates a Deployment record in etcd database
- API then sends a broadcasting event saying a new nginx deployment is created
- Deployment Controller receives the event from API server, it then requests the API server to create a replicaset for the nginx deployment
- API Server receives the request from Deployment Controller, it then creates a ReplicaSet record in etcd database
- API Server sends a broadcasting event saying a new nginx replicaset is created
- ReplicaSet Controller receives the event from API Server, it then requests the API Server via REST call to create 3 Pods
- API Server create 3 Pods entries(records) in etcd database
- API Server sends a broadcasting event saying new Pod created. This kind of event will be broadcasted for every Pod.
- Scheduler receives the event, it then identifies a healthy node where the new Pod can be deployed. Scheduler sends its scheduling recommendataions to API Server via REST Call
- API Server, retrieves the existing Pod entries from etcd database and it updates the Pod with the new scheduling information
- API Server then sends broadcasting events saying Pod scheduled to so and so node(s), this would be repeated for every Pod
- In case a Pod is scheduled to worker1, the kubelet service running in worker will receive the event from API Server, it then checks if the respective container image is present in the local node, if not will request CRI-O container runtime to download the image. Once the image is download, kubelet creates the containers required by the Pod with the respective container image. 
- kubelet monitors the status of the containers running locally, it then frequently like a heart-beat it keeps updating the status to API Server via REST calls
- API server receives the kubelet status updates, it then updates the Pod status in the etcd database
</pre>


## Info - Deployment Controller
<pre>
- When pods are requested by the user, it will try to spread the pods on multiples nodes but there is no assurance
- it is possible all the pods from a specific deployment may be scheduled to the same nodes as well
- Deployment controller doesn't put any constraint on scheduling, hence it is upto the scheduler to decide which pod goes to which node
- what is the guarantee offered 
  - at point of time, the desired number of pods will be always running
  - but they can run on any nodes
</pre>

## Info - DaemonSet Controller
<pre>
- unlike the Deployment Controller, daemonset controller ensures one Pod per node are deployed
- the number of Pods deployed will be equivalent to the number of nodes in your openshift cluster
- hence, the pods created as part of daemonset are distributed always one Pod per node
- What is the practical usecase for this?
  - prometheus pod to collect performance metrics need to run in every node
  - kube-proxy pod in Kubernetes/Openshift runs in every node
- what is the guarantee offerred
  - if 5 nodes are there in cluster, 5 pods would be created
  - each Pod would be scheduled to different nodes in the cluster
  - the total number of pods and nodes would match 
</pre>

## Info - StatefulSet Controller
<pre>
- used to deploy stateful applications
- they tend to use external storage in general
- generally when we have deploy database applications as a cluster that synchronizes data
- creating a cluster of database varies for every database, hence cluster creation is our responsibility
- statefulset provides the required sections/provisions to create a cluster, but it won't create a cluster of databases out of the box
- scaling up/down the number of db pods and make them work as cluster is very complex, hence we also need to do some configurations to ensure they are running as a cluster
</pre>

## Info - Job Controller
<pre>
- is used to do any one time activity
- the one time activity will run in a container which of part of a Pod
- example
  - taking one time backup of etcd database
</pre>  

## Info - CronJob Controller
<pre>
- is used to do any repeating tasks 
- it can be scheduled to run on a particular day/week/month and particular time
- example
  - taking backup, every friday midnight
</pre>

## Info - Deployment
<pre>
- Stateless application are installed into openshift as Deployment
- Under each deployment there would be atleast one ReplicaSet
- Under each ReplicaSet, there could be 0 to n number of Pods
- Every ReplicaSet, manages one version of docker image, in other words for each version of your application one replicaset will be created
</pre>  

## Info - Labels
<pre>
- key/values pairs
- labels are used to map child objects
- every components has one to many labels
- the deployment will identify its respective replicaset using labels as a selector
- the replicaset will identify its respective pods using labels as a selector
- For example
  - Each Deployment has one or more ReplicaSets
  - Each ReplicaSet has one or more Pod
  - Each Pod has one or more Containers
</pre>

## Lab - Create a project to seggregate your deployments from other users
```
oc new-project jegan
oc get projects
oc get projects | grep jegan
```

Expected output
![image](https://github.com/user-attachments/assets/9a2964e9-aadd-479c-9f27-0ecee9e63e8e)
![image](https://github.com/user-attachments/assets/77263d5c-b4fb-4aac-a1f3-ef3dab135e03)

## Lab - Deploying our first application into openshift
First check which is active currently, make sure you are deploying in your project
```
oc project
oc create deployment nginx --image=bitnami/nginx:latest --replicas=3
```

Listing the deployments
```
oc get deployments
oc get deployment
oc get deploy
```

Listing the replicasets
```
oc get replicasets
oc get replicaset
oc get rs
```

Listing all pods
```
oc get pods
oc get pod
oc get po
```

Expected output
![image](https://github.com/user-attachments/assets/e103e971-684e-4c4f-a355-251dc2a50887)
![image](https://github.com/user-attachments/assets/93a893da-4840-4792-9baf-56b5416a1a96)
![image](https://github.com/user-attachments/assets/a946d031-eac6-4e0f-8e44-b7d3785689f2)
![image](https://github.com/user-attachments/assets/74517c7b-47c0-4593-9406-47d0e79e1935)

Listing multiple openshift objects with a single get command
```
oc get deploy,rs,po
```

Expected output
![image](https://github.com/user-attachments/assets/8a365632-6513-49c2-bd49-836318e1947a)

## Lab - Find more details about a pod
```
oc get po
oc describe pod/nginx-66c775969-l9tqs
```

Expected output
![image](https://github.com/user-attachments/assets/532f291c-42df-4fb5-8730-0b62e17900a7)
![image](https://github.com/user-attachments/assets/e23fde80-5647-4286-afeb-a47b8833a7ca)
![image](https://github.com/user-attachments/assets/a8c7d9a0-5d7d-42b3-aac1-db0d1abe94fe)

## Lab - Finding more details about a replicaset
```
oc get rs
oc describe rs/nginx-66c775969
```

Expected output
![image](https://github.com/user-attachments/assets/151a3aa0-d9b4-446f-94e3-3a349462631b)
![image](https://github.com/user-attachments/assets/8f9d7e24-b26a-4bac-aad4-e65fb97fc7fb)

## Lab - Finding more details about a deployment
```
oc get deploy
oc describe deploy/nginx
```

Expected output
![image](https://github.com/user-attachments/assets/e7a7ae7e-e57a-41fe-9c03-c48e728e86f2)
![image](https://github.com/user-attachments/assets/57fbdc49-7648-4a6a-a63b-40e9b0246914)

## Lab - Scaling up nginx deployment from 3 pods to 5 and scale down to 2 pods
```
oc get deploy,rs,po
oc scale deploy/nginx --replicas=5
oc get po
oc scale deploy/nginx --replicas=2
oc get po
```

Expected output
![image](https://github.com/user-attachments/assets/05db61b9-fa43-4e6c-b004-cf37df6f4f3e)

## Lab - Editing an existing deployment
```
oc get deploy
oc edit deploy/nginx
```

Expected output
![image](https://github.com/user-attachments/assets/3fca651b-0fee-40dc-9e02-739d26415d23)
![image](https://github.com/user-attachments/assets/419fa67a-1d86-42a8-a7ad-1bf1c3df5083)

## Lab - Editing a replicaset
```
oc get rs
oc edit rs/nginx-66c775969
```
Expected output

![image](https://github.com/user-attachments/assets/8a269bee-6b55-4bea-9891-c21919c4fa5b)
![image](https://github.com/user-attachments/assets/16b9fdf8-efb4-4660-94ee-435f135501b6)
![image](https://github.com/user-attachments/assets/52588a8c-366f-4244-b112-788449d5befd)
![image](https://github.com/user-attachments/assets/3a836d7e-d5ad-4be2-8fcc-e9a0aa0f3599)

## Lab - Editing a pod
```
oc get po
oc edit po/ngix-66c775969-psvt9
```

Expected output
![image](https://github.com/user-attachments/assets/cbc6493a-ee58-42c8-846c-fa1331357f62)
![image](https://github.com/user-attachments/assets/7fd1953f-fb8b-4036-a778-20be1422f3eb)


## Lab - Find the pod id and node where it is running
```
oc get po -o wide
```

Expected output
![image](https://github.com/user-attachments/assets/3f7c5346-8fcd-4654-8b86-50ec09f2a7cb)

## Lab - Getting inside a pod shell
```
oc get po
oc rsh pod/<pod-name> sh
oc rsh -c <container-name> pod/<pod-name> sh
oc rsh deploy/nginx 
```

Expected output
![image](https://github.com/user-attachments/assets/057e1764-29f5-4aae-899b-8b6e6eec8a1d)

## Lab - Port forwarding - should never be used in production ( it is meant for developer testing )
In the below command, port 9091 is the localhost port,while 8080 is the port where the nginx  web server is listening on the container.  Instead of port 9091, you could any port of your choice that is available on your local machine.
```
oc get po
oc pod-forward pod/<pod-name> 9091:8080
```

From another terminal tab, you may try accessing the web page
```
curl http://localhost:9091
```

Expected output
![image](https://github.com/user-attachments/assets/41c48fcf-3261-4c2f-955b-26eb8ca098ee)
![image](https://github.com/user-attachments/assets/a322f8ba-ce46-401d-a056-a23a97d41a61)

## Info - Openshift service
<pre>
- service represents a group of load-balanced pods
- the pods behind a service comes from a single deployment
- when the deployment is scaled up/down the service - pod endpoints are updated automatically
- no two pods should communicate with each other directly, they should always communicate via a service
- accessing a service by its name is called service discovery
- openshift has dns service
- service discovery only works within the cluster
- kube-proxy - provides load-balancing
</pre>

## Lab - Creating an external nodeport service for deployment nginx
```
oc get deploy
oc expose deploy/nginx --type=NodePort --port=8080
oc get services
oc get service
oc get svc
oc describe svc/nginx
```

Expected output
![image](https://github.com/user-attachments/assets/b3900278-1b68-4de3-ac77-8464bc072821)
![image](https://github.com/user-attachments/assets/a8ff83e1-c434-4f58-af8b-c0870ca1fd7f)

Accessing a NodePort service from outside the cluster
```
oc get nodes -o wide
curl http://<node-ip>:<node-port>
```

Expected output
![image](https://github.com/user-attachments/assets/e046a7d8-dae6-4739-9000-f7b24fd43f1f)
![image](https://github.com/user-attachments/assets/72ed87bd-12de-4a5e-987f-1495fdedccc7)

