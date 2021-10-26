### 1 - Choose the environment to test the Operator

Test the Operator by deploying it on the Local Kubernetes provided out of the box. The test will deploy the Operator and all the Kinds.

Goto the **Test** tab. Select the **Use Default Kubernetes** option. Click **Deploy**.



### 2 - Deploy the Operator



### 3 - Check Operator status

Wait for a ~30 seconds for the deployment to complete.

Click **Check Operator Status** to get the status of the Operator.



### 4 - Verify the Operator is deployed from the Kubernetes console

Operator is deployed  in the **[operator name]-system** namespace. 

```bash
kubectl get deployment -n nginx-operator-system
```

Get the resources installed by the CRD. It should give the resources selected while creating the Operator. 

```
kubectl get pod,svc,configmap,secret -n nginx-operator-system
```

Alternately get the URL of the application and check from the browser.

```
echo "http://<ip>:$(kubectl get service nginxsvc -n nginx-operator-system -o custom-columns=:spec.ports[0].nodePort | tail -1)"
```

### 5 - Undeploy the Operator

Click **Undeploy** to un-install the Operator and the CRD.

### 6 - Verify the Operator is Undeployed from the Kubernetes console

Check if the namespace created as part of the test still exists.

```
kubectl get namespace nginx-operator-system
```



