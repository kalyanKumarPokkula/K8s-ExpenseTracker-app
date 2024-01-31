# Cloud-Native web ExpenseTracker Application with Kubernetes

---

This cloud-native web application is built using a mix of technologies. It's designed to be accessible to users via the internet, allowing them to Signup and add a expense update and delete the expenses

## Tech Stack

- [Frontend:](The frontend of this application is built using React and JavaScript. It provides a and user-friendly interface for expensetracker.)

- [Backend-and-API:](The backend of this application is powered by nodejs. It serves as the API handling user expense requests. MongoDB is used as the database backend, configured with a replica set for data redundancy and high availability.)

#### Once the Cluster is ready run the command to set context

```
aws eks update-kubeconfig --name EKS_CLUSTER_NAME --region NAME_OF_REGION_THAT_CLUSTER_CREATED

```

To check the nodes in your cluster run

```
Kubectl get nodes

```

Clone the github repo

```
git@github.com:kalyanKumarPokkula/K8s-ExpenseTracker-app.git

```

##### Create Cloudchamp Namespace

```
kubectl create ns cloudchamp

kubectl config set-context --current --namespace cloudchamp

```

##### Mongo Database Setup

To create a Mongo statefulset with Persistent volumes, run the command in manifest folder:

```
kubectl apply -f mongodb-statefulset.yaml

```

Mongo Service

```
kubectl apply -f mongodb-headless-service.yaml

```

On the mongo-0 pod, initialise the Mongo database Replica set. In the terminal run the following command:

```
kubectl exec -it mongo-0 -- mongo
rs.initiate();

rs.add("mongo-1.mongo:27017");

rs.add("mongo-2.mongo:27017");

cfg = rs.conf();

cfg.members[0].host = "mongo-0.mongo:27017";

rs.reconfig(cfg, {force: true});


```

To set mongo-1.mongo-2 as secondary use this command it is one process

```
kubectl exec -it mongo-1 -- mongosh

db.getMongo().setReadPref("secondary)

kubectl exec -it mongo-2 -- mongosh

db.getMongo().setReadPref("secondary)


```

##### API Setup

Create a expensetracker-api-deployment by running the following command

```
kubctl apply -f expensetracker-deployment.yaml

```

Expose expensetracker-api-deployment through service using the following command:

```
kubectl apply -f expensetracker-service.yaml

```

To conform the whether expensetracker-api-service ready to recieve HTTP traffic. In the terminal run the following command:

```
{

API_ELB_PUBLIC_FQDN=$(kubectl get svc expensetracker-api-service -ojsonpath="{.status.loadBalancer.ingress[0].hostname}")
until nslookup $API_ELB_PUBLIC_FQDN >/dev/null 2>&1; do sleep 2 && echo waiting for DNS to propagate...; done
curl $API_ELB_PUBLIC_FQDN
echo
}

```

Add the url of expensetracker-api-service loadbalancer to a frontend expensetracker-ui-deployment.yaml

```
env:
    - name: REACT_APP_API_URL
      value: Add the url of expensetracker-api-service
      loadbalancer
```

##### Frontend setup

Create the Frontend Deployment resource. In the terminal run the following command:

```
kubectl apply -f expensetracker-ui-deployment.yaml

```

Create a new Service resource of LoadBalancer type. In the terminal run the following command:

```
kubectl apply -f expensetracker-ui-service.yaml

```

Confirm that the Frontend ELB is ready to recieve HTTP traffic. In the terminal run the following command:

```
{
FRONTEND_ELB_PUBLIC_FQDN=$(kubectl get svc expensetracker-ui-service -ojsonpath="{.status.loadBalancer.ingress[0].hostname}")
until nslookup $FRONTEND_ELB_PUBLIC_FQDN >/dev/null 2>&1; do sleep 2 && echo waiting for DNS to propagate...; done
curl -I $FRONTEND_ELB_PUBLIC_FQDN
}

```

## Summary

---

In this Project, you learnt how to deploy a cloud native application into EKS. Once deployed and up and running, you used your local workstation's browser to test out the application. You later confirmed that your activity within the application generated data which was captured and recorded successfully within the MongoDB ReplicaSet back end within the cluster.
