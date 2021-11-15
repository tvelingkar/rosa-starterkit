### 1 - Create the namespace

```execute
kubectl create ns pgo
```

### 2 - Create the secret

Create the secret with Database configuration
```execute
kubectl create secret generic postgres-config --from-literal=POSTGRES_DB=contacts --from-literal=POSTGRES_USER=pguser --from-literal=POSTGRES_PASSWORD=password -n pgo
```

### 3 - Create the Persistent Volumes
Create Persistent Volume and Persistent Volume Claims.

```execute
cat <<EOF>volume.yaml
    kind: PersistentVolume
    apiVersion: v1
      metadata:
        name: postgres-pv-volume
        labels:
          type: local
          app: postgres
      spec:
        storageClassName: manual
        capacity:
          storage: 500Mi
        accessModes:
         - ReadWriteMany
        hostPath:
          path: "/mnt/data"
      ---
      
      kind: PersistentVolumeClaim
      apiVersion: v1
      metadata:
        name: postgres-pv-claim
        namespace: pgo
        labels:
          app: postgres
      spec:
        storageClassName: manual
        accessModes:
         - ReadWriteMany
           resources:
               requests:
           storage: 100Mi
EOF
```

```execute
kubectl apply -f volume.yaml
```         
      
### 4 - Create the Database

Create the deployment that creates the Database instance.
```execute
cat <<EOF>deployment.yaml
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        labels:
          app: postgres
        name: postgres
        namespace: pgo
      spec:
        replicas: 1
        selector:
          matchLabels:
            app: postgres
        strategy: {}
        template:
          metadata:
            labels:
              app: postgres
          spec:
            containers:
            - image: postgres:10.19
              name: postgres
              resources: {}
              ports:
                  - containerPort: 5432
              envFrom:
                  - secretRef:
                      name: postgres-config
              volumeMounts:
                  - mountPath: /var/lib/postgresql/data
                    name: postgredb
            volumes:
              - name: postgredb
                persistentVolumeClaim:
                  claimName: postgres-pv-claim
EOF
```

```execute
kubectl apply -f deployment.yaml
```     
      
### 5 - Create the Database conectivity

Create the service to communicate with the Database.
```execute
cat <<EOF>service.yaml
      apiVersion: v1
      kind: Service
      metadata:
        name: contacts
        namespace: pgo
        labels:
          app: postgres
      spec:
        type: NodePort
        ports:
         - port: 5432
           selector:
              app: postgres
EOF
```
      
```execute
kubectl apply -f service.yaml
```  

### 6 - Verify the Database setup 
Get the Cluster IP:

```execute
export ip_addr=$(ifconfig eth1 | grep inet | awk '{print $2}' | cut -f2 -d:)
```

Check the Cluster IP Address:

```execute
echo $ip_addr
```


```execute
port=$(kubectl get svc -n pgo -o custom-columns=:spec.ports[0].nodePort | tail -1)
export PGPASSWORD=$(kubectl get secret postgres-config -n pgo -o=jsonpath='{.data.POSTGRES_PASSWORD}' | base64 --decode)
echo $PGPASSWORD
nc -z -v -w30 $ip_addr $port
psql -U postgres -h $ip_addr -p $port contacts
```


```execute
\q
```



### 5 - Start creating the Operator

This tutorial explains creating an Ansible Operator from the Kubernetes setup provided out of the box with this Lab.
Open the application.
![CreateOperator1](/Users/shraddhaparikh/OpGenerator/GitHub/rosa-starterkit/_images/CreateOperator1.png)

### 6 - Select the Method of creating Operator

Choose the Operator Method as **Ansible Operator from Existing Kubernetes Resources**

![OperatorMethod](/Users/shraddhaparikh/OpGenerator/GitHub/rosa-starterkit/_images/OperatorMethod.png)

### 7 - Give the operator details and fetch the resources

Give the name of the Operator and details as below.

Since we are fetching from the Kubernetes provided out of the box, select **Use local Kubernetes** option and click button **Use local Kubernetes**.

Give the namespace from where resources are to be fetched. For this lab give the namespace as **local-k8s-setup**. Click button **Fetch resources** .



### 1 - Download the Operator Code

Goto the **Submit and Download** tab. Click the **Download** button.

![Download1](/Users/shraddhaparikh/OpGenerator/GitHub/rosa-starterkit/_images/Download1.png)

### 2 - Alternate method

Goto the main page. Click the download icon of the Operator.

![](/Users/shraddhaparikh/OpGenerator/GitHub/rosa-starterkit/_images/Download2.png)
