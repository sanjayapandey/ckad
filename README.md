#CKAD Cheatsheet


###############################
kubectl
###############################

kubectl <command> --help
kubectl explain object.sub-object
e.g kubectl explain pod.spec.containers
e.g kubectl explain deploy 

####################
Node
####################

kubectl get no
kubectl get no -o wide
kubectl describe no <node-name>

#####################################
Pod
#####################################
kubectl get po
kubectl get po -o wide 
kubectl get po <pod name> -o yaml # output in yaml format
kubectl get po <pod name> -o json # output in json format
kubectl describe po <pod name>
kubectl delete po <pod name>
kubectl delete po --all # deletes all pods
kubectl apply -f <pod.yaml> -n <namespace-name>

# If you are not given a pod definition file but a pod, you may extract the definition to a file using the below command.
# Then edit the file to make the necessary changes, delete and re-create the pod.
kubectl get po <pod-name> -o yaml > pod.yaml
vi pod.yaml
kubectl delete po <pod-name>
kubectl apply -f pod.yaml

# To edit pod properties
kubectl edit po <pod-name> # opens in vi editor

kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
kubectl run hazelcast --image=hazelcast/hazelcast --labels="app=hazelcast,env=prod"
kubectl run hazelcast --image=hazelcast/hazelcast --env="DNS_DOMAIN=cluster" --env="POD_NAMESPACE=default"
kubectl run nginx --image=nginx -- <arg1> <arg2> ... <argN>
kubectl run nginx --image=nginx --command -- <cmd> <arg1> ... <argN>


# To execute a command inside a running container in the pod

kubectl exec [POD] -- [COMMAND]
kubectl exec nginx -- ls /
kubectl exec -it nginx -- /bin/sh # Interactive mode. Get a shell to the container running in your Pod
kubectl exec -it web -c nginx -- /bin/bash 

# In a multicontainer pod, containers are created and destroyed together. 

# Just like containers, initContainers is a property under spec and it has same sub properties like name, image etc.

# Note: kubectl run command creates the pod and not deployment.
# There is no imperative command like kubectl create po for pod creation.

# name, labels, annotations and namespace are part of metadata.
# kubectl get po -A  # all pods in all namespaces
# kubectl get events -A | grep error # all events in all namespaces with errors
# Pods have a property called restartPolicy whose values can be Always, Never or OnFailure.
  Default value is Always.
# To add the labels , just pass the value after -l flag
k run messaging —image: redis -l tire=message
# delete pods with label 
k delete pod -l name=busy-box


#############################
commands and arguments
#############################

# A container only lives as long as the process inside it is alive. If the process inside the container is finished or crashes, the container exits.
# When you specify the command in JSON array format, the first element in the array should be executable. e.g command: ["sleep","4800"]
# To define a command, include the command field in configuration file.
# To define arguments for the command, include the args field in configuration file.
# The command and arguments that you define in the pod configuration file override the default command and arguments provided by the container image.
# If you define args, but do not define a command, the default command is used with your new arguments.
# In a docker file, cmd provides default argument to entrypoint instruction.
# command replaces the image's ENTRYPOINT instruction, the command that is executed to start your container.
# args replaces the image's CMD instruction. This list of arguments is passed to the command specified in the previous field.


# Example1:
spec:
  containers:
  - name: command-demo-container
    image: debian
    command: ["printenv"]
    args: ["HOSTNAME", "KUBERNETES_PORT"]
	
#       command:
        - "sleep"
        - "5000"
	

# Example3: To run your command in a shell
command: ["/bin/sh"]
args: ["-c", "while true; do echo hello; sleep 10;done"]

# Example4: 
command: ["sleep","5000"]	

# Example7:
command: ["ls", "index.html"]

Note: Enter the command as is in a shell form. Or in a JSON array format (Preferred). In a JSON array format, the first element should be executable.

# 
cat Dockerfile2
FROM python:3.6-alpine
RUN pip install flask
COPY . /opt/
EXPOSE 8080
WORKDIR /opt
ENTRYPOINT ["python", "app.py"]
CMD ["--color", "red"]

On container startup, the following command would run:
python app.py --color red


cat Dockerfile
FROM python:3.6-alpine
RUN pip install flask
COPY . /opt/
EXPOSE 8080
WORKDIR /opt
ENTRYPOINT ["python", "app.py"]

On container startup, the following command would run:
python app.py 

#
If you supply a command but no args for a Container, only the supplied command is used. 
The default ENTRYPOINT and the default CMD defined in the Docker image are ignored.

#
If you supply a command and args, the default ENTRYPOINT and the default CMD defined in the Docker image are ignored. 
Your command is run with your args.

#
If you supply only args for a Container, the default Entrypoint defined in the Docker image is run with the args that you supplied.
For more details, you can refer to this test:
https://kodekloud.com/courses/certified-kubernetes-administrator-with-practice-tests-labs/lectures/12038798

###################################
replicaset
###################################
kubectl apply -f <replicaset-definition.yaml>
kubectl get rs
kubectl get rs -o wide
kubectl delete rs <replicaset-name>
kubectl scale rs <> --replicas=6 
or
kubectl edit rs <replicaset-name>
kubectl describe rs <replicaset-name>

# There are 2 ways to edit a rs.
Either delete and re-create the ReplicaSet or 
Update the existing ReplicaSet and then delete all PODs, so new ones with the correct image will be created.

kubectl edit rs <rs-name>
kubectl delete po <rs-po-name> # Delete all pods under rs.

kubectl get rs <rs-name> -o yaml > rs.yaml
vi rs.yaml
kubectl apply -f rs.yaml
kubectl delete po <rs-po-name> # Delete all pods under rs.

# The value for labels in spec.selector clause and spec.template.metadata should match in replicaset.

# To see all the objects at once in current namespace
kubectl get all

# Replicaset has a template.
# if there are any other pods matching the matchLabels , it will not create a new pod and tries to match the total number of replicas. If you already have replicaset running and 
# trying to run new pod with the same labels, it will immediately turns down since it will always tries to math the total number of running pod having the same labels. 
##########################
Deployments
##########################
kubectl create deploy nginx --image=nginx --replicas=2
kubectl get deploy
kubectl get deploy -n <namespace-name> 
kubectl get deploy <deployment-name>
kubectl get deploy <deployment-name> -o yaml | more
kubectl get deploy -o wide
kubectl describe deploy <deployment-name>
kubectl set image deployment/<deployment-name> <container-name>=<new-image>
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
# .spec.replicas is an optional field and it’s default value is 1

kubectl apply -f deploy.yaml 
kubectl apply -f deploy.yaml --record # to record the change-cause in revision history
kubectl rollout status deploy <deployment-name> # when you run the create or deploy command, it will create the rollout as well 
# Deployment’s rollout is triggered if and only if the pod template inside .spec.template is changes, for example if the label or container images of the template is updated
# other update like scaling the deployment will not trigger a rollout 
kubectl rollout history deploy <deployment-name>
kubectl rollout history deploy nginx --revision=2 # to get detailed history for a specific revision.
kubectl rollout undo deploy <deployment-name> 
kubectl rollout undo deploy <deployment-name> --to-revision=1
# Pause the Deployment to apply multiple fixes and then resume it to start a new rollout. ( otherwise, on each update on deployment It will start the auto deploy and records the changes)
#When the deployment is paused, no changes are recorded to revision history.
# you can’t rollback the paused deployment until you resume it 
kubectl rollout pause deploy <deployment-name> 
kubectl rollout resume deploy <deployment-name>
kubectl delete deploy <deployment-name>
# The value for labels in selector clause and pod template should match in deployment.
kubectl create deploy nginx --image=nginx --dry-run=client -o yaml > deploy.yaml
kubectl scale deploy <deployment-name> --replicas=3 # To scale up / down a deployment. Not recorded in revision history.
kubectl create deploy <deployment-name> --image=redis -n <namespace-name>
# RollingUpdateStrategy in deployment details define upto how many pods can be down/up during the update at a time.

kubectl edit deploy <deployment-name> -n <namespace-name> # to make changes to a running deployment. Another option is to update the yaml file and apply it.

# Properties to remember
spec.strategy.type==RollingUpdate|Recreate
spec.strategy.rollingUpdate.maxUnavailable
spec.strategy.rollingUpdate.maxSurge

#
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0 #We can give a direct number (eg, 5) or percentage of desired pods ( eg , 10%), default is 25%. This ensures that at least 75% pods are up
      maxSurge: 1 #We can give a direct number (eg, 5) or percentage of desired pods ( eg , 10%), default is 25%. Total no of old and new pods does not exceed 125% of desired pods
	  
# keys follow camel case naming convention. values follow initials in upper case convention.

# Deployment has a template.

# Whenever you create a new deployment or update images in an existing deployment, a rollout is triggered. 
# A new rollout creates a new deployment revision.

# You can locate a problematic deployment in cluster by checking the READY status for deployments. The no of containers running will be less than desired.
# A deployment automatically creates a replicaset which in turn creates pods.

# To donate the no of seconds the deployment controller waits before indicating that deployment progress has stalled 
kubectl patch deployment/nginx-deployment -p '{"spec":{"progressDeadlineSeconds":600}}'
# Default revision history is 10. If we explicitly set the value to 0, will result in cleaning all the history of deployment, so Deployment will not be able to rollback 
.spec.revisionHistoryLimit

#########################
Deployment strategy 
#########################
# Deployment Strategy : Blue Green 
There will be a BLUE application already running with the current version. The service definition file will have the selection with version: v1 
The new version v2 will be deployed using different deployment file called Green. 
Once we verify that the new version ( blue deployment) is working as expected, we now update the service definition file selector level to v2

# Deployment Strategy: Canary  
We only direct small percentage traffic ( eg, 25%) to new version and verify .
We will have primary deployment running with the pre-defined number of replicas. 
Now, create a new deployment called myapp-canary.yaml with same labels but the only required replicas ( eg, 1 replica). 
Since we have the same labels, service starts to route traffic to the new pods and old pods based on defined load balancing algorithm. 
After monitoring the newer version for some time, we will decrease the number of replicas on old version and increase replicas on newer version. 

# You can add new label to differentiate different releases 
# You can tweak the number of replicas of the stable and canary releases to determine the ratio of each release that will receive the live traffic 

#########################
HorizontalPodAutoScaler
#########################
 #If HorizontalPodAutoScaler is managing scaling for a deployment, don’t set .spec.replicas 

kubectl get hpa
kubectl delete hpa <hpa-name>
kubectl autoscale deploy nginx --min=5 --max=10 --cpu-percent=80
kubectl autoscale deployment/nginx-deployment --min=10 --max=15 --cpu-percent=80


####################
service
####################
kubectl apply -f <svc.yaml>
kubectl get svc
kubectl describe svc <service-name> # to get to know about port, target port etc.
kubectl delete svc <service-name>
# Copy the label from template section of deployment and paste it under selector section of service.
# If you are creating a frontend service for enabling external access to users, set the type to NodePort.
# If you are tasked to create a service to enable the frontend pods to access a backend set of pods, set the type to ClusterIP.
# In ClusterIP, port refers to port on service aka port exposed by service and targetPort refers to port on pod (container) aka port exposed by container.
# LoadBalancer service type only works with supported cloud platforms like GCP, AWS, Azure. In an unsupported environment like VirtualBox, it would have the same effect as setting it to NodePort.
kubectl expose pod redis --port=6379 --name=redis-service --type=ClusterIP --dry-run=client -o yaml > svc.yaml
kubectl expose pod nginx --port=80 --target-port=8080 --name=nginx-service --type=NodePort --dry-run=client -o yaml > svc.yaml
kubectl expose deploy <deploy-name> --port=<> 
# Note: We can't set nodePort using imperative command. So for node port services, use yaml instead of command.

# To get endpoints for a service
kubectl get ep <svc-name> # endpoint is nothing but the Pods to which the service is linked. It contains ip of pod and port.
kubectl edit svc <svc-name> # to edit a service
# services, deployments, replicasets and network policy have selector property under spec for selecting pods using labels.
# There are 3 ports involved - node port, port and target port. node port is port on the node.
  port is port on service. target port is port on pod. port is mandatory. 
  IP of service is known as Cluster-IP (internal ip).
  Service can be accessed by pods using Cluster-IP or service name.

# create a service to expose a deployment in certain namespace within cluster on port 
kubectl expose deployment redis --port=6379 --name messaging-service --namespace marketing

# expose the deployment using cluster ip 
kubectl expose deployment redis --name=redis --port=6379 --target-port=6379
 
################ 
namespace
################
kubectl get ns
kubectl create ns <namespace-name>
kubectl apply -f <namespace.yaml>
kubectl get po -n kube-system
kubectl describe ns <namespace-name>
kubectl delete ns <namespace-name> 
# To switch to a namespace permanently
kubectl config set-context $(kubectl config current-context) --namespace=dev
# To view pods in all namespaces
kubectl get po -A
# To create a pod in a specific namespace
kubectl run redis --image=redis -n <namespace-name> 
kubectl exec -it <pod-name> -n <namespace-name> -- sh
kubectl get all -A # To check all objects in all namespaces
kubectl create ns <namespace-name> --dry-run=client -o yaml 

######################
ConfigMap
######################
kubectl get cm
kubectl describe cm <cm-name> # to check key-value pairs in config map
kubectl create cm <cm-name> --from-file=<path to file> # colon or equals to as delimiter between keys and values
kubectl create cm <cm-name> --from-file=<directory>
kubectl create cm <cm-name> --from-literal=<key1>=<value1> --from-literal=<key2>=<value2>
kubectl apply -f cm.yaml

# Properties to remember:
configmapkeyref / env, configMapRef / envFrom, volume

# Pods can consume ConfigMaps as environment variables or as configuration files in a volume mounted on one or more of its containers for the application to read.
  When ConfigMap is created from a file (kubectl create cm <cm name> --from-file= ) and when that ConfigMap is mounted as volume, 
  then the entire file is available at mount point for the pod.

# Injected into the Pod.

######################
Secrets
######################
kubectl get secrets
kubectl describe secret <secret-name> # This shows the attributes in secret but hides the values.
kubectl get secret <secret-name> -o yaml # To view the values (encoded).
# If you enter the pod where secret is injected, you can see decoded values.
kubectl create secret generic <secret-name> --from-literal=<key1>=<value1> --from-literal=<key2>=<value2>
kubectl create secret generic <secret-name> --from-file=<path to file> # colon or equals to as delimiter between keys and values

# Properties to remember:
secretkeyref / env, secretref / envFrom, volume

# A secret can be injected into a pod as file in a volume mounted on one or more of its containers or as container environment variables.

# opaque and service-account-token are secret types.

# For encoding, decoding
echo -n 'string' | base64
echo -n 'encoded string' | base64 --decode

# While creating secret with the declarative approach (yaml), you must specify the secret key and value in encoded format.
# When we create secret using imperative approach, secret keys and values are encoded on their own (and decoded as well).

# Injected into the Pod.

# https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/learn/lecture/14827414#overview


######################
Service Account
######################

kubectl create sa <sa-name>
kubectl get sa
kubectl describe sa <sa-name>

# To fetch token from service account
kubectl describe sa <sa-name> # gives secret name
kubectl describe secret <secret-name> # gives token stored in secret

# Create an nginx pod that uses 'myuser' as a service account
kubectl run nginx --image=nginx --serviceaccount=myuser --dry-run=client -o yaml > pod.yaml
kubectl apply -f pod.yaml

# When we use service account inside the pod, the secret for that service account is mounted as volume inside the pod.

# Property to remember: spec -> serviceAccountName # set at pod level
# Injected into the Pod.
# For a deployment, can set service account in pod template.

# A user makes a request to API server through kubectl using user account.
# A process running inside a container makes a request to API server using service account.
  A service account just like user account has certain permissions.

# we can retrieve the token of the service account by running following command
kubectl create service account <service-account-name>
kubectl create token <service-account-name> # It will valid for 1 hours


######################
Taints (on nodes) and Tolerations (on pods)
######################
# To check taints on a node
kubectl describe no <node01> | grep -i "taint"
# To create a taint on node01 with key of 'spray', value of 'mortein' and effect of 'NoSchedule'.
# Taint effect defines what happens to pods that do not tolerate this taint. 
# Values for taint effect are NoSchedule, NoExecute and PreferNoSchedule.
kubectl taint no node01 spray=mortein:NoSchedule 
# The property tolerations under spec has properties like key, operator, value and effect and their values come inside "".
# Remove the taint on master, which currently has the taint effect of NoSchedule
kubectl taint no master node-role.kubernetes.io/master:NoSchedule-
# Remove from node 'foo' all the taints with key 'dedicated'
kubectl taint no foo dedicated-


#######################
Logging
#######################
# The standard output of a container can be seen using the logs command.
kubectl logs -f <pod-name> <container-name> # follow the logs 
kubectl logs <pod-name> --previous # dump pod logs for a previous instantiation of a container


#######################
Monitoring
#######################

kubectl top no
kubectl top po
# To get name of pod that is consuming most CPU.
kubectl top pod --namespace=default | head -2 | tail -1 | cut -d " " -f1
kubectl top po --sort-by cpu --no-headers


#######################
Jobs
#######################
kubectl create job busybox --image=busybox -- /bin/sh -c "echo hello;sleep 30;echo world"
kubectl get jobs
kubectl logs busybox-qhcnx # pod under job
kubectl delete job <job-name>
# In case of pods, default value for restart property is Always and in case of jobs, default value for restart property is Never.
# Job has pod template. Job has 2 spec sections - one for job and one for pod (in order).
# A pod created by a job must have its restartPolicy be OnFailure or Never. If the restartPolicy is OnFailure, a failed container will be re-run on the same pod. If the restartPolicy is Never, a failed container will be re-run on a new pod.

# Job properties to remember: completions, backoffLimit, parallelism, activeDeadlineSeconds, restartPolicy.
# Job will run uninterrupted unless the pod fails ( restartPolicy=Never) or container exits ( restartPolicy=OnFailure), at which point job defers to the .spec.backoffLimit. Once .spec.backoffLimit has been reached, the job will marked as failed and running pods will be terminated. 
# .spec.activeDeadlineSeconds takes precedence over .spec.backoffLimit. Once it reached the time limit specified by activeDeadlineSeconds, it will terminated. 
# By default, pods in a job are created one after the other (sequence). Second pod is created only after the first one is finished.

#######################
CronJobs
#######################

kubectl get cj
# Create a cron job with image busybox that runs on a schedule and writes to standard output
kubectl create cj busybox --image=busybox --schedule="*/1 * * * *" -- /bin/sh -c "date; echo Hello from Kubernetes cluster" 
# In a cronjob, there are 2 templates - one for job and another for pod.
# In a cronjob, there are 3 spec sections - one for cronjob, one for job and one for pod (in order).
# Properties to remember: spec -> successfulJobHistoryLimit, spec -> failedJobHistoryLimit


#######################
Ingress
#######################
kubectl get ingress  # To check details about Ingress Resource
kubectl describe ingress <>
kubectl edit ingress <ingress name> 
kubectl apply -f <ingress.yaml>
# Ingress setup requires an ingress controller (deployment), a node port ingress service (for accessing ingress controller from outside the cluster) and a config map.
  They all are in same namespace.
# The ingress resource (type is ingress), application(deployment) and service (for accessing deployment) are in different namespace.
# In order for the Ingress resource to work, the cluster must have an ingress controller running.
  Unlike other types of controllers which run as part of the kube-controller-manager binary, Ingress controllers are not started automatically with a cluster.
  Kubernetes supports AWS, GCE and nginx ingress controllers.
# Ingress resource defines rules and Ingress controller fulfills those rules.

kubectl create ingress <name> —rule=host/path=service:port[,tls[=secret]]
eg: kubectl create ingress sample —rule=“<service-name>/bar=svc1:8080, tls=my-cert”
eg: kubectl create ingress myapp --class=default —rule=“mydaomain.com/=svc:port” —rule=“mydaomain.com/admin/=svcadmin:portadmin” 
eg: kubectl create ingress ingress —rule=“mock-exam.com/video*=my-video-service:8080” 
# To create a ingress
spec.backend.serviceName: <name_of_service_to_route_traffic>
spec.backend.servicePort: <service_port>	

# There are 3 ways to route traffic 
# Create the ingress file and have the following properties 
  1. Domain name or Host based 
spec: 
   rules: 
   - host: vides.netflix.com
      http: 
        paths: 
        - path: /vides
           backend: 
               serviceName: <>
               servicePort: <>           
     - host: purchasee.netflix.com
         http:
            paths: 
2. Route based : To route traffic based on route (eg. /videos, /purchase) 
spec: 
   rules: 
   - http: 
        paths: 
        - path: /vides
           backend: 
               serviceName: <>
               servicePort: <>           
        - path: /purchase

# user the default route to show 404 page
https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-ingress-em- 

#######################
Volumes
#######################
kubectl get pv
kubectl get pvc
kubectl delete pvc <pvc-name>
kubectl delete pv <pv-name>

Properties to remember:
spec -> volumes -> name: 
                   emptyDir: {}
				   
spec -> containers -> volumeMounts -> name:
                                      mountPath: 
									  

spec -> volumes -> name: 
                   hostPath -> type: Directory
				               path: 
				   
spec -> containers -> volumeMounts -> name:
                                      mountPath:	

# pvc is to be injected into the pod. Pods use pvc as volume and pod's containers mount that volume.
# pvc remains in pending state until it is bound to a pv.
# storageClassName and accessModes must match between pv and pvc. If no storage class is specified in PV, then there should be no storage class in PVC as well.
  storage size must also be in range.			
# persistent volumes have reclaim policy whose values are recycle (data in volume to be purged), retain (data and volume to be retained) and delete (volume to be deleted).
  reclaim policy is invoked when pvc is deleted. once pvc is deleted, future of pv depends on reclaim policy.
# property is spec -> persistentVolumeReclaimPolicy.		
# PVs use labels and PVCs use selectors for selecting the PVs.		
# PVs are cluster wide and PVCs are namespaced.	  

#######################
Labels
#######################
kubectl get po|deploy|all --show-labels
kubectl label po nginx env=lab # Add a label to a pod
kubectl label deploy my-webapp tier=frontend # Add a label to a deployment
kubectl label no node01 size=large # To label nodes

kubectl label po nginx env- # Remove the label
kubectl label po nginx env=lab1 --overwrite # to overwrite a label
# Duplicate keys can't be used in labels.


#######################
Selectors
#######################
kubectl get po --selector=app=App1
kubectl get po --selector=app!=App1
kubectl get all --selector=env=prod
kubectl get po --selector=env=prod,bu=finance,tier=frontend # equivalent of && in programming languages

######################
Environment Variables
#######################
kubectl run nginx --image=nginx --env=app=web # Create an nginx pod and set an environment variable 
# env and envFrom property is an array.
# env property takes two properties - name and value. value takes only string and will always come in double quotes.


#######################
Annotations
#######################
kubectl annotate po nginx desc="Hello World"
kubectl annotate po nginx author=Sanjaya
kubectl annotate po nginx desc- # Remove this annotation from the pod
# Duplicate keys in annotations are not allowed.

#######################
Security
#######################
cat /etc/kubernetes/manifests/kube-apiserver.yaml
# Role based authentication 
# Create a Role with roles.api group/resources/verbs 
rules:
- apiGroups: [“”] resources: [“pods”,”configMap”] verbs: [“get”,”create”,”update”]
# Create a RoleBinding subject.kind/name/apiGroup and roleRef.kind/name/apiGroup 

kubectl get roles 
kubectl gete rolebindings
kubectl describe role <role-name>

# To check the access 
kubectl auth can-i create deplyment 
kubectl auth can-i create deployments —as dev-user —namespace prod 



#######################
Security Context
#######################
# Properties to remember: securityContext -> runAsUser and securityContext -> capabilities -> add
Note: securityContext -> capabilities -> add property takes array as value. For example:	
add:
- "NET_ADMIN"
- "SYS_TIME"
 
OR

add: ["NET_ADMIN", "SYS_TIME"]

Note: runAsUser takes only numeric values.

# Can be set at pod level as well as container level. 
If you configure at pod level, the security settings will carry over to all the containers with in the pod.
If you configure at both pod level and container level, the settings on container will override the settings on pod.

#######################################
requests and limits | Resource Quotas
#######################################
# We can set quotas for the total amount of memory and cpu that can be consumed by all the containers running in a namespace.
  Quotas are specified using ResourceQuota object.
# Once ResourceQuota is there, Every Container must have a memory request, memory limit, cpu request, and cpu limit.
# We can configure default memory requests and limits for a namespace using LimitRange object in namespace. 
  If a Container is created in a namespace that has a default memory limit, and the Container does not specify its own memory limit, then the Container is assigned the default memory limit.
# We can configure default CPU requests and limits for a namespace using LimitRange object. 
  If a Container is created in a namespace that has a default CPU limit, and the Container does not specify its own CPU limit, then the Container is assigned the default CPU limit.
# Requests and Limits can be set for each of the containers in the Pod. If not set, they take default values.
kubectl run nginx --image=nginx --requests="cpu=100m,memory=256Mi" --limits="cpu=200m,memory=512Mi"
# cpu can also be specified as 1,2,0.1 etc. Here 1 count of cpu is equivalent to 1 vCPU in AWS or 1 GCP core. 0.1 count = 100m where m is milli.
  It can go as low as 1m.
# minimum usage through requests and maximum usage through limits.
  Requests are what the container is guaranteed to get. If a container requests a resource, Kubernetes will only schedule it on a node that can give it that resource. 
  Limits, on the other hand, make sure a container never goes above a certain value. The container is only allowed to go up to the limit, and then it is restricted.
# Resource quota is specified at namespace level. Resource limits (requests and limits) are specified at container level.
# If a pod tries to exceed resources beyond its specified limit, then in case of cpu, Kubernetes throttles cpu. 
  A container can't use more cpu resources than its limit. But a container can use more memory resources than its limit.
  If a pod tries to use more memory resources than its limit constantly, pod will be terminated.
kubectl get quota
kubectl describe quota -n <namespace-name>
# Good explaination in Golden book.


#######################	 
Network Policy
#######################
kubectl get netpol
kubectl describe netpol <name>
Note: While creating network policy, make sure that not only network policy is applied to the correct object but also that it allows access from (ingress) / to correct object (egress).
labels and selectors are used.
An empty podSelector selects all pods in the namespace.

#######################
Probes
#######################
# Set at the level of containers 
# readinessProbe is to check whether the application is ready.
# livenessProbe is to check whether the application is live (running).

Properties to remember: 
livenessProbe -> httpGet -> port
                         -> path
						
livenessProbe -> exec -> command

livenessprobe -> tcpSocket -> port 

livenessProbe -> initialDelaySeconds

livenessProbe -> PeriodSeconds

livenessProbe -> failureThreshold


readinessProbe -> httpGet -> port
                         -> path
						
readinessProbe -> exec -> command

readinessprobe -> tcpSocket -> port 

readinessProbe -> initialDelaySeconds

readinessProbe -> PeriodSeconds

readinesProbe -> failureThreshold

#############################################################################################################
Notes:
1.) securityContext -> runAsUser is available at pod as well as container level.
    securityContext -> capabilities -> add only available at container level.
2.) volumeMounts has properties like name, mountPath and readOnly (true|false)
4.) Deployment doesn't have good example in kubernetes.io, so remember this:
spec:
 replicas: 5
 strategy: 
  type: RollingUpdate
  rollingUpdate:
   maxSurge: 7
   maxUnavailable: 3
5.) The labels in pod template and selector clause in deployment must match.
6.) If you are using a port in readiness probe or liveness probe, then that port must be exposed.
7.) If unit for cpu request is given as 200m then use that ("200m"). If it is given as .2, then use that ("0.2")
8.) name in volumes and volumeMounts must be same.
9.) env is used with configMapKeyRef. In doc, search by configMapKeyRef. name property of env is used as such. Used to import one key from configmap.
    envFrom is used with configMapRef. In doc, search by configMapRef. Used to import the entire configmap.
	When configmap is mounted as volume, then no need to think of env or envFrom. Just think of volumes and volumeMounts.
10.) env is used with secretKeyRef. In doc, search by secreyKeyRef. name property of env is used as such. Used to import one key from secret.
     envFrom is used with seretRef. In doc, search by secretRef. used to import the entire secret.
	 When secret is mounted as volume, then no need to think of env or envFrom. Just think of volumes and volumeMounts. 
11.) To check connectivity, you can use curl or netcat.
nc -v -w 2 -z ip port
curl http://ip:port


