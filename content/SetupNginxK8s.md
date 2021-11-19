### 1 - Create the namespace

```execute
kubectl create ns local-k8s-setup
```

### 2 - Create the secret and configmaps

Create self signed SSL certificate and key to access the application securely. Press enter for all the field values.

```execute
openssl req -nodes -new -x509 -keyout key.pem -out cert.pem -days 365
```

Create tls secret from the cert and key

```execute
kubectl create secret tls nginxsecret --key key.pem --cert cert.pem  -n local-k8s-setup
```

Create the nginx configuration file 

```execute
cat <<EOF>nginx.conf

server {
        listen 80 default_server;
        listen [::]:80 default_server ipv6only=on;

        listen 443 ssl;
    
        root /usr/share/nginx/html;
        index index.html;
    
        server_name localhost;
        ssl_certificate /etc/nginx/ssl/tls.crt;
        ssl_certificate_key /etc/nginx/ssl/tls.key;
    
        location / {
                try_files $uri $uri/ =404;
        }
}
EOF
```

Create a Configmap for the nginx configuration file.

```execute
kubectl create configmap nginxconfigmap --from-file=nginx.conf -n local-k8s-setup
```

Create the index.html.

```execute
cat <<EOF>index.html
<!DOCTYPE html>
<html>
	<head>
	<title>OpenLabs StarterKit Tutorial</title>
	<style>
  body {
  width: 35em;
  margin: 0 auto;
  font-family: Tahoma, Verdana, Arial, sans-serif;
  }
	</style>
	</head>
	<body>
  <h1>Kubernetes and OpenShift StarterKit Tutorial</h1>
	<p>Thank you for going through the Starter Kit Tutorial.</p>
	</body>
</html>
EOF
```

Create a Configmap for the index.html file.

```execute
kubectl create configmap indexconfigmap --from-file=index.html -n local-k8s-setup
```

### 3 - Deploy the application

Create the yaml file to deploy the service and nginx.

```execute
cat <<EOF>nginxapp.yaml
---
apiVersion: v1
kind: Service
metadata:
  name: nginxsvc
  labels:
    app: nginx
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
    name: http
  - port: 443
    protocol: TCP
    name: https
  selector:
    app: nginx
---
apiVersion: v1
kind: Pod
metadata:
  name: starterkit-nginx
  labels:
    app: nginx
spec:
  volumes:
  - name: secret-volume
    secret:
       secretName: nginxsecret
  - name: configmap-volume
    configMap:
       name: nginxconfigmap
  - name: index-volume
    configMap:
       name: indexconfigmap
  containers:
  - name: nginxhttps
    image: nginx
    ports:
    - containerPort: 443
    - containerPort: 80
    volumeMounts:
    - mountPath: /etc/nginx/ssl
      name: secret-volume
    - mountPath: /etc/nginx/conf.d
      name: configmap-volume
    - mountPath: /usr/share/nginx/html
      name: index-volume
EOF
```

Create the service and application

```execute
kubectl create -f nginxapp.yaml -n local-k8s-setup
```

### 4 - Verify the setup is complete

```execute
kubectl get svc,pod,configmap,secret -n local-k8s-setup
```

Check the URL in the browser

```execute
echo "http://$(hostname -I | cut -d' ' -f2):$(kubectl get service nginxsvc -n local-k8s-setup -o custom-columns=:spec.ports[0].nodePort | tail -1)"
```
We have completed the setup of the application for which we want to create an Operator. The remaining steps explain how to use the StarterKit to create an Operator from this setup.

### 5 - Start creating the Operator

This tutorial explains creating an Ansible Operator from the Kubernetes setup provided out of the box with this Lab.
Open the application.
![CreateOperator1](../_images/CreateOperator1.png)

### 6 - Select the Method of creating Operator

Choose the Operator Method as **Ansible Operator from Existing Kubernetes Resources**

![OperatorMethod](../_images/OperatorMethod.png)

### 7 - Give the operator details and fetch the resources

Give the name of the Operator and details as below.

Since we are fetching from the Kubernetes provided out of the box, select **Use local Kubernetes** option and click button **Use local Kubernetes**.

Give the namespace from where resources are to be fetched. For this lab give the namespace as **local-k8s-setup**. Click button **Fetch resources** .

 

![OperatorDetails](../_images/OperatorDetails.png)

The **Add Kind +** option on the left panel will be enabled only if the resources are fetched successfully.

### 8 - Create a Kind using the K8s resources

Add a new CRD(Kind) using **Add Kind +**

Give the kind name. Select the resources that will make up the CRD.

![Kinddetails](../_images/Kinddetails.png)

More than 1 Kind can be added.

### 9 - Create the Operator

Goto the Submit & Download tab.

Click **Submit Operator** to create the Operator.

![Submit](../_images/Submit.png)

If the Operator is created successfully, you will see a message as below.

![SubmitSuccessful](../_images/SubmitSuccessful.png)



### 10 - Choose the environment to test the Operator

Test the Operator by deploying it on the Local Kubernetes provided out of the box. The test will deploy the Operator and all the Kinds.

Goto the **Test** tab. Select the **Use Default Kubernetes** option. Click **Deploy**.

![Deploy](../_images/Deploy.png)

### 11 - Deploy the Operator

![DeploySuccessful](../_images/DeploySuccessful.png)



### 11 - Check Operator status

Wait for a ~30 seconds for the deployment to complete.

Click **Check Operator Status** to get the status of the Operator.

![StatusSuccessful](../_images/StatusSuccessful.png)

### 12 - Verify the Operator is deployed from the Kubernetes console

Operator is deployed  in the **[operator name]-system** namespace. 

```execute
kubectl get deployment -n nginx-operator-system
```

Get the resources installed by the CRD. It should give the resources selected while creating the Operator. 

```execute
kubectl get pod,svc,configmap,secret -n nginx-operator-system
```

Sample output-

![resourcescreated](../_images/resourcescreated.png)

Alternately get the URL of the application you just deployed using the Operator and check from the browser.

```execute
echo "http://$(hostname -I | cut -d' ' -f2):$(kubectl get service nginxsvc -n nginx-operator-system -o custom-columns=:spec.ports[0].nodePort | tail -1)"
```

### 13 - Undeploy the Operator

Click **Undeploy** to un-install the Operator and the CRD.

![Undeployed](../_images/Undeployed.png)

### 14 - Verify the Operator is Undeployed from the Kubernetes console

Check if the namespace created as part of the test still exists.

```execute
kubectl get namespace nginx-operator-system
```

### 15 - Download the Operator Code

Goto the **Submit and Download** tab. Click the **Download** button.

![Download1](../_images/Download1.png)

### 16 - Alternate method

Goto the main page. Click the download icon of the Operator.

![](../_images/Download2.png)