### Deployments

- Get the documentation of `Deployments` and its fields.

```bash
kubectl explain deployments
```

- Create yaml file named `mydeployment.yaml` and explain fields of it.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    environment: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

- Create the deployment with `kubectl apply` command.
  
```bash
kubectl apply -f mydeployment.yaml
```

- List the deployments.

```bash
kubectl get deployments
```

- List pods with more information.
  
```bash
kubectl get pods -o wide
```

- Show details of deployments.

```bash
kubectl describe deploy/nginx-deployment
```

- Print the logs for a container in a pod.

```bash
kubectl logs <pod-name>
```

- If there is a multi-container pod, we can print logs of one container.

```bash
kubectl logs <pod-name> -c <container-name>
```

- Execute a command in a container.

```bash
kubectl exec <pod-name> -- date
```

```bash
kubectl exec <pod-name> -- cat /usr/share/nginx/html/index.html
```

- Open a bash shell in a container.

```bash
kubectl exec -it <pod-name> -- bash
```

- List the ReplicaSets.

```bash
kubectl get rs
```

- Show details of ReplicaSets.

```bash
kubectl describe rs <rs-name>
```

- Scale the deployment up to five replicas.

```bash
kubectl scale deploy nginx-deployment --replicas=5
```

- But each time do we have to apply these commands for scaling? No because there will be our yml file and we can change it when we need scale.

>> Show when you apply mydeployment.yaml change, how differ?

- Delete a pod and show new pod is immediately created.

```bash
kubectl delete pod <pod-name>
kubectl get pods
```

- Delete deployments

```bash
kubectl delete deploy <deployment-name>
```
## Part 3 - Deployment Rolling Update and Rollback in Kubernetes

- Create a new folder name it deployment-lesson.

```bash
mkdir deployment-lesson
cd deployment-lesson
```

- Create a `clarus-deploy.yaml` and input text below. Pay attention that image version is 1.0.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: clarus-deploy
  labels:
    app: container-info
  annotations:
    kubernetes.io/change-cause: deploy/clarus-deploy is set as container-info=clarusway/container-info:1.0
spec:
  replicas: 3
  selector:
    matchLabels:
      app: container-info
  template:
    metadata:
      labels:
        app: container-info
    spec:
      containers:
      - name: container-info
        image: clarusway/container-info:1.0
        ports:
        - containerPort: 80
```

- Create the deployment with `kubectl apply` command.

```bash
kubectl apply -f clarus-deploy.yaml
```

- List the `Deployment`, `ReplicaSet` and `Pods` of `clarus-deploy` deployment using a label and note the name of ReplicaSet.

```bash
kubectl get deploy,rs,po -l app=container-info
```

- Describe deployment and note the image of the deployment. In our case, it is clarusway/container-info:1.0.

```bash
kubectl describe deploy clarus-deploy
```

- View previous rollout revisions.

```bash
kubectl rollout history deploy clarus-deploy
```

- Display details with revision number, in our case, is 1. And note name of image.

```bash
kubectl rollout history deploy clarus-deploy --revision=1
```

- Upgrade image.

```bash
kubectl set image deploy clarus-deploy container-info=clarusway/container-info:2.0
```

- Show the rollout history.

```bash
kubectl rollout history deploy clarus-deploy
```

- Display details about the revisions.

```bash
kubectl rollout history deploy clarus-deploy --revision=1
kubectl rollout history deploy clarus-deploy --revision=2
```

- List the `Deployment`, `ReplicaSet` and `Pods` of `clarus-deploy` deployment using a label and explain ReplicaSets.

```bash
kubectl get deploy,rs,po -l app=container-info
```

- Upgrade image with kubectl edit commands.

```bash
kubectl edit deploy/clarus-deploy
```

- We will see an output like below.

```yaml
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "2"
    kubectl.kubernetes.io/last-applied-configuration: |
    ...
```

- Change the `metadata.annotations.kubernetes.io/change-cause` and `spec.template.spec.containers.image` fields as below.

```yaml
...
...
    kubernetes.io/change-cause: kubectl set image deploy clarus-deploy container-info=clarusway/container-info:3.0
...
...
    spec:
      containers:
      - image: clarusway/container-info:3.0
...
...
```

- Show the rollout history.

```bash
kubectl rollout history deploy clarus-deploy
```

- Display details about the revisions.

```bash
kubectl rollout history deploy clarus-deploy --revision=1
kubectl rollout history deploy clarus-deploy --revision=2
kubectl rollout history deploy clarus-deploy --revision=3
```

- Apply `kubectl get rs` and show how many replica set exist and explain why?

- List the `Deployment`, `ReplicaSet` and `Pods` of `clarus-deploy` deployment using a label and explain ReplicaSets.

```bash
kubectl get deploy,rs,po -l app=container-info
```

- Rollback to `revision 1`.

```bash
kubectl rollout undo deploy clarus-deploy --to-revision=1
```

- Show the rollout history and show that we have revision 2, 3 and 4. Explain that original revision, which is `revision 1`, becomes `revision 4`.

```bash
kubectl rollout history deploy clarus-deploy
kubectl rollout history deploy clarus-deploy --revision=2
kubectl rollout history deploy clarus-deploy --revision=2
kubectl rollout history deploy clarus-deploy --revision=4
```

- Try to pull up the `revision 1`, that is no longer available.

```bash
kubectl rollout history deploy clarus-deploy --revision=1
```

- List the `Deployment`, `ReplicaSet` and `Pods` of `mynginx` deployment using a label, and explain that the original ReplicaSet has been scaled up back to three and second ReplicaSet has been scaled down to zero.

```bash
kubectl get deploy,rs,po -l app=container-info
```

- Delete the deployment.

```bash
kubectl delete deploy -l app=container-info
```