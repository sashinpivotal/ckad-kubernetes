# CKAD resources

- [https://github.com/twajr/ckad-prep-notes#detailed-review](https://github.com/twajr/ckad-prep-notes#detailed-review)
- [https://www.avthart.com/posts/certified-kubernetes-exam-tips/](https://www.avthart.com/posts/certified-kubernetes-exam-tips/)
- [Mock exercises](https://github.com/dgkanatsios/CKAD-exercises)
- [Eitan](https://eitansuez.github.io/writeups/ckad-retrospective.html)


# vi

- [linux academy tutorial](https://linuxacademy.com/blog/linux/vi-short-cuts-for-beginners/)
- [https://blog.codonomics.com/2019/09/essential-vim-for-ckad-or-cka-exam.html](https://blog.codonomics.com/2019/09/essential-vim-for-ckad-or-cka-exam.html)

```
'd$' - delete the rest of the line
'dG' - Deletes contents from cursor to end of file. This is very useful when editing YAML files.
'ZZ' - Save and exit quickly.
Insert & Paste - In order to copy from other app, Insert mode and then Cmd+V
`dd' 
`o` - add a new line
```

- I can I display line number in vi? Type "/" and line number
- how to delete a line? dd
- How do I set the default editor?
- vim available? yes

# Nano

```text
export KUBE_EDITOR=nano
  - F9 - delete a line
  - F6 - search
  - ESC-U - undo?? (M-U)
  - CTRL+E - end of a line
  - CTRL+A - start of the line
  - ESC+G - go to a line
  - ?? How do copy and paste in nano
```

# Chrome

```text
- Shortcut keys
  - Fn+Left to the top page
  - Fn+Right to the bottom page
  - ?? How to get the search box?  There is none.
  - Shift+Space one page up
  - Space one page down
```

# General kubectl commands

- Delete all resources from current namespace

```
k delete all --all
```

- Make sure the env values are with double-quote

- Setting namespaces

```
kubectl config set-context --current --namespace=mynamespace
```

- *Creating a yaml file from cut and paste from kubernetes.io

```text
$cat > my.yaml
(copy and paste - move the curson to the next line)
CTRL+C
```

- You have to use -- /bin/sh -c 'multiple commands'
- **You have to use --dry-run and -o yaml before --

```
k create cronjb my-cronjob --image=busybox --schedule="* * * * *" --dry-run -o yaml  -- /bin/sh -c 'date; echo Hello from the Kubernetes cluster'  
```

- *What is the following for? It is to create deployment yaml file.

```
$kubectl create deployment my-deploy --image=nginx --dry-run=client -o yaml > deployment.yaml
```

- *Is the following for creating deployment? It is
 to create a service from a deployment

```text
$kubectl expose deployment my-deploy --name=my-service --target-port=8080 --port 80 --dry-run -oyaml > my.yaml
```

- Cert exam tips

```text
While you would be working mostly the declarative way - using definition files, imperative commands can help in getting one time tasks done quickly, as well as generate a definition template easily. This would help save a considerable amount of time during your exams.

Before we begin, familiarize with the two options that can come in handy while working with the below commands:

--dry-run: By default as soon as the command is run, the resource will be created. If you simply want to test your command, use the --dry-run option. This will not create the resource, instead, tell you whether the resource can be created and if your command is right.

-o yaml: This will output the resource definition in YAML format on the screen.

Use the above two in combination to generate a resource definition file quickly, that you can then modify and create resources as required, instead of creating the files from scratch.
```

```text
POD -Create an NGINX Pod

kubectl run nginx --restart=Never --image=nginx
kubectl run mypod --restart=Never --image=nginx  --dry-run -o yaml
```

```text
IMPORTANT: How to create deployment and scale it

kubectl create deployment does not have a --replicas option. You could first create it and then scale it using the kubectl scale command.

Save it to a file - (If you need to modify or add some other details)

kubectl create deployment --image=nginx nginx --dry-run -o yaml > nginx-deployment.yaml

You can then update the YAML file with the replicas or any other field before creating the deployment.

Or you can scale it using
k scale --replicas=3 deployment.apps/my-deploy

Or you can run it with --replicas=3
k run my-deploy --image=nginx --replicas=3
```

```text
How to create a service from a pod
Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379

kubectl expose pod redis --port=6379 --name redis-service --dry-run -o yaml

(This will automatically use the pod's labels as selectors)
```

Or

```text
kubectl create service clusterip redis --tcp=6379:6379 --dry-run -o yaml  (This will not use the pods labels as selectors, instead it will assume selectors as app=redis. You cannot pass in selectors as an option. So it does not work very well if your pod has a different label set. So generate the file and modify the selectors before creating the service)
```

Create a Service named nginx of type NodePort to expose pod named nginx with port 80 on port 30080 on the nodes:

kubectl expose pod nginx --port=80 --name nginx-service --dry-run -o yaml

(This will automatically use the pod's labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod.)

Or

kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run -o yaml

(This will not use the pods labels as selectors)

Both the above commands have their own challenges. While one of it cannot accept a selector the other cannot accept a node port. I would recommend going with the `kubectl expose` command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.

Reference:

https://kubernetes.io/docs/reference/kubectl/conventions/

- Getting list of all resources 

```
kubectl api-resources -o wide
```

# Pod & Deployment

- Create a pod imperatively, note -l option for label

```bash
k run  my-pod --restart=Never --image=redis:alpine -l tier=msg
```

- Create a deployment - this done does not work in Katacoda
  (It works in docker desktop v 1.16)

```
k run my-webapp --image=nginx --labels=tier=frontend --replicas=3
```

- For creating a yaml from an existing pod - Then edit the file to make the necessary changes, delete and re-create the pod.

```
k get pod my-pod -o yaml > my.yaml
```

- Or edit it but when you save it, it will create a temporary
  file name starting with /tmp/xxx, or you can save it 
  `:w my.yaml`. Then you will have to delete -f and crete -f
  with the file

```
k edit pod my-pod. 
```

- Attach to a running pod

```
< sang/tmp > k exec -it nginx -- ls /
bin    dev    home   media  opt    root   sbin   sys    usr
data   etc    lib    mnt    proc   run    srv    tmp    var
```

- Here is what [CrashLoopBackoff error means](https://sysdig.com/blog/debug-kubernetes-crashloopbackoff/)


## env

### Trouble-shooting

- *You will experience the following problem when the `env.value`
  is not quoted with double-quote

```
< 1pivotal/ckad > k create -f my3.yaml
Error from server (BadRequest): error when creating "my3.yaml": Pod in version "v1" cannot be handled as a Pod: v1.Pod.Spec: v1.PodSpec.Containers: []v1.Container: v1.Container.Env: []v1.EnvVar: v1.EnvVar.Value: ReadString: expects " or n, but found 8, error found in #10 byte of ...|,"value":8080}],"ima|..., bigger context ...|e":"tails"},{"env":[{"name":"NGINX_PORT","value":8080}],"image":"nginx","name":"sonic"}]}}
|...
```

  So do as following:
  
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: sega
  name: sega
spec:
  containers:
  - image: busybox
    name: tails
    command: ["sleep"]
    args: ["3600"]
  - image: nginx
    name: sonic
    env:
      - name: NGINX_PORT
        value: "8080"  # make sure the value is double quoted
```

- *You get the following error

```
Invalid value: "The edited file failed validation": ValidationError(Pod.spec.containers[0].env): invalid type for io.k8s.api.core.v1.Container.env: got "map", expected "array"
```
  When you missed `-` for `name` below
  
```
 23   containers:
 24   - image: nginx
 25     env:
 26       name: MY_ENV. # Error on this line
 27       value: "100"
```

# Replica set

- *(mock exam question) Fix the original replica set 'new-replica-set' to use the correct 'busybox' image
Either delete and re-create the ReplicaSet or Update the existing ReplicSet and then delete all PODs, so new ones with the correct image will be created.

```
kubectl edit rs new-replica-set
```

Make sure to delete all pods and recreate by

```
k delete -f <yaml file>
k create -f <yaml file>

or you can
l apply -f <yaml file>
```

# Service

```bash
kubectl expose pod redis --port=6379 --name redis-service
```

- Create a service named as ingress with NodePort type (the
  default is `ClusterIP`

```bash
kubectl expose deployment my-deployment --type=NodePort --port=80 --name=ingress --dry-run -o yaml >ingress-service.yaml
```

- Make sure to use `target-port` instead of `targetPort`

```
< sang/tmp > k expose pod messaging --port=81 --target-port=8081 --name=my-messaging-service
```

- *If you don't specify the `name=`, the default name of
  the service is the name of the resource, from which the
  service gets created

```
< sang/tmp > k expose pod messaging --port=81 --target-port=8081
Error from server (AlreadyExists): services "messaging" already exists
```

- *It automatically creates Cluster-IP (which is the default)

```
< sang/tmp > k get svc messaging
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
messaging   ClusterIP   10.106.164.148   <none>        81/TCP    4m44s
```

# Deployment

```
k create deployment webapp --image=kodekloud/webapp-color
```

```
kubectl scale deployment/webapp --replicas=3
```

# ConfigMap

```
k create cm my-config --from-literal=DB_Host=my-host --from-literal=DB_Port=2333
```



## Secret

- *Make sure to add `generic` when creating a secret - 
  make sure to
  place `generic` before the name of the secret

- *The second works.  The environment variable name is 
  case-sensitive

```
master $ k create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root  --from-literal=DB_Password=password123
secret/db-secret created
```

- Example of working pod definition with a secret

```
master $ cat  /var/answers/answer-webapp.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: webapp-pod
  name: webapp-pod
  namespace: default
spec:
  containers:
  - image: kodekloud/simple-webapp-mysql
    imagePullPolicy: Always
    name: webapp
    envFrom:      # access value indirectly
    - secretRef:
        name: db-secret
```

or through a volume

```
master $ cat  /var/answers/answer-webapp.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: webapp-pod
  name: webapp-pod
  namespace: default
spec:
  containers:
  - image: kodekloud/simple-webapp-mysql
    imagePullPolicy: Always
    name: webapp
  volumes:
  - name: my-secret-volume. # each volume has to have a name
    secret:
      secretName: db-secret
```

- Try to find the syntax using explain

```
k explain pod.spec.containers.envFrom
```

# SecurityContext

# ReadinessProbe, LivenessProbe

```
Update the newly created pod 'simple-webapp-2' with a readinessProbe using the given spec
Spec is given on the right. Do not modify any other properties of the pod.
```

```yml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/liveness
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
```

# Jobs

```
Create a Job using this POD definition. Look at how many attempts does it take to get a '6'.
Use the specification given on the right.
```

```bash
k create job throw-dice-job --image=kodekloud/throw-dice
```

```bash
# Create a job with command
kubectl create job my-job --image=busybox -- date
```

# Ingress

- *Is there a way to create an ingress in imperative way?  You can use the following service and
you will have to modify it - no the best way is to get sample
ingress from kube io and modify

```text
(Mockexam) You are requested to make the new application available at /pay.
Identify and implement the best approach to making this application available on the ingress controller and test to make sure its working. Look into annotations: rewrite-target as well.

info_outline
Hint
Create a new Ingress for the new pay application in the new namespace. Check out the answer at /var/answers
Ingress Created
Path: /pay
Configure correct backend service
Configure correct backend port
```

- *How do you create ingress resource imperatively?
  (answer) You can't.  You have to create yaml from the k8s site
  such as following. You have to find out the port
  number of the service.

```yml
master $ cat /var/answers/ingress-pay.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /pay
        backend:
          serviceName: pay-service
          servicePort: 8282
```

- *ingress does not get displayed with all

```
master $ k get ingress --all-namespaces
NAMESPACE   NAME                 HOSTS   ADDRESS   PORTS   AGE
app-space   ingress-wear-watch   *                 80      9m43s
```

```
master $ cat /var/answers/ingress-resource.yaml
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear-watch
  namespace: app-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /wear
        backend:
          serviceName: wear-service
          servicePort: 8080
      - path: /watch
        backend:
          serviceName: video-service
          servicePort: 8080
```

# NetworkPolicy

- *What is the problem of the following yaml file?
  There was `ports:` missing

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
  namespace: default
spec:
  egress:
  - to:
    - podSelector:
        matchLabels:
          name: payroll
    ports:
    - port: 8080
      protocol: TCP
    to:
    - podSelector:
        matchLabels:
          name: mysql
    - port: 3306
      protocol: TCP
  podSelector:
    matchLabels:
      name: internal
  policyTypes:
  - Egress
```

- The following is the Correct one

```
master $ cat  /var/answers/answer-internal-policy.yaml
```

```yml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: internal-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      name: internal
  policyTypes:
  - Egress
  - Ingress
  ingress:
    - {}
  egress:
  - to:
    - podSelector:
        matchLabels:
          name: mysql
    ports:
    - protocol: TCP
      port: 3306

  - to:
    - podSelector:
        matchLabels:
          name: payroll
    ports:
    - protocol: TCP
      port: 8080

  - ports:
    - port: 53
      protocol: UDP
    - port: 53
      protocol: TCP
```

- *You would experience the following error

```
< 1pivotal/ckad > k create -f my2.yaml
error: error validating "my2.yaml": error validating data: ValidationError(NetworkPolicy.spec.ingress[0].from[0].podSelector.matchLabels): invalid type for io.k8s.apimachinery.pkg.apis.meta.v1.LabelSelector.matchLabels: got "string", expected "map"; if you choose to ignore these errors, turn validation off with --validate=false
```

when you have - there was no space between `access:` and `redis`

```
< 1pivotal/ckad > cat my2.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: redis-access
  namespace: default
spec:
  podSelector:
    matchLabels:
      name: redis
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          access:redis
```

# Mock exercises

```
Apply a label app_type=beta to node node02. Create a new deployment called beta-apps with image:nginx and replicas:3. Set Node Affinity to the deployment to place the PODs on node02 only


NodeAffinity: requiredDuringSchedulingIgnoredDuringExecution
```

```
Update pod app-sec-kff3345 to run as Root user and with the SYS_TIME capability.
```

