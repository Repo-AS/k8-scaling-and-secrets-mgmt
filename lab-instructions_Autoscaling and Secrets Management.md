::page{title="Practice Lab: Autoscaling and Secrets Management"}

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/images/IDSN-logo.png" alt="cognitiveclass.ai logo" width="200" />  
  

This practice lab is designed to provide hands-on experience with Kubernetes, focusing on vertical and horizontal pod autoscaling and secrets management.

## Objectives

In this practice lab, you will:

-   Build and deploy an application to Kubernetes
    
-   Implement Vertical Pod Autoscaler (VPA) to adjust pod resource requests/limits
    
-   Implement Horizontal Pod Autoscaler (HPA) to scale the number of pod replicas based on resource utilization
    
-   Create a Secret and update the deployment for using it
    

> > **Note: Kindly complete the lab in a single session without any break because the lab may go in offline mode and cause errors. If you face any issues/errors during the lab process, please logout from the lab environment. Then, clear your system cache and cookies and try to complete the lab.**

::page{title="Setup the environment"}

On the menu bar, click `Terminal` and select the `New Terminal` option from the drop-down menu.

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/labs/1_ContainersAndDocker/images/env&cmdlinetools_1.png" width="800" />  

> Note: If the terminal is already open, please skip this step.

### Step 1: Verify kubectl version

Before proceeding, ensure that you have kubectl installed and properly configured. To check the version of kubectl, run the following command:

```
kubectl version

```

You should see the following output, although the versions may be different: ![kubectl-version.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/-NFh46Dm4FZOHXnmQFui7Q/kubectl-version.png)

### Step 2: Clone the project repository

Clone the repository with the starter code to commence the project.

```
git clone https://github.com/ibm-developer-skills-network/k8-scaling-and-secrets-mgmt.git

```

::page{title="Exercise 1: Build and deploy an application to Kubernetes"}

The Dockerfile in this repository already has the code for the application. You are just going to build the docker image and push it to the registry.

You will be giving the name `myapp` to your Kubernetes deployed application.

### Step 1: Build the Docker image

1.  Navigate to the project directory.
    

```
cd k8-scaling-and-secrets-mgmt

```

2.  Export your namespace.
    

```
export MY_NAMESPACE=sn-labs-$USERNAME 

```

3.  Build the Docker image.
    

```
docker build . -t us.icr.io/$MY_NAMESPACE/myapp:v1 

```

![build-N2.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/D3b7KoSAtaA6lJKaLVTCRw/build-N2.png)

### Step 2: Push and list the image

1.  Push the tagged image to the IBM Cloud Container Registry.
    

```
docker push us.icr.io/$MY_NAMESPACE/myapp:v1 

```

![docker-push-N2.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cuRDiJbujY7CdcA_pXCU2w/docker-push-N2.png)

2.  List all the images available. You will see the newly created `myapp` image.
    

```
ibmcloud cr images

```

![image-listing.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/c50-gVLc8V6pCFg_5Rfdmg/image-listing.png)

### Step 3: Deploy your application

1.  Open the `deployment.yaml` file located in the main project directory. It's content will be as follows:
    

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  labels:
    app: myapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - image: us.icr.io/<your SN labs namespace>/myapp:v1
        imagePullPolicy: Always
        name: myapp
        ports:
        - containerPort: 3000
          name: http
        resources:
          limits:
            cpu: 50m
          requests:
            cpu: 20m

```

2.  Replace `<your SN labs namespace>` with your actual SN lab's namespace.
    

<details>
<summary>Click here for the ways to get your namespace</summary>
<ol><li><p>Run the command <code>oc project</code>, and use the namespace mapped to your project name
<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/-3mH40uvqVfBAqGnP3JIJQ/geting-nmspce-1.png" alt="geting-nmspce-1.png"></p></li><li><p>Run the command <code>ibmcloud cr namespaces</code> and use the one which shows your sn-labs-username
<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/ezM2h2TpCG-43_vSPOB18A/getting-nmspce-2.png" alt="getting-nmspce-2.png"></p></li></ol>
</details>

3.  Apply the deployment.
    

```
kubectl apply -f deployment.yaml

```

![deployment-applied.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/s4_tuXcGmL1rWyFZ76-iUw/deployment-applied.png)

4.  Verify that the application pods are running and accessible.
    

```
kubectl get pods

```

![depl-pods.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/2rW3Cp3-9wDXT-_NfxAjIA/depl-pods.png)

### Step 4: View the application output

1.  Start the application on port-forward:
    

```
kubectl port-forward deployment.apps/myapp 3000:3000 

```

![port-fwd-N2.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/2FdOB0jSTtJNvimvutgbIQ/port-fwd-N2.png)

2.  Launch the app on Port `3000` to view the application output.
    
    1.  Click on the Skills Network Toolbox Icon.
        
    2.  Click on Launch Application.
        
    3.  Enter the Port Number.
        
    4.  Click the launch button as shown in the screenshot.
        

![deploy-getdjango-1](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/zx6iQ8coZ1SIMZUSwtG8zA/deploy-getdjango-1.jpg)

3.  You should see the message `Hello from MyApp. Your app is up!`.
    

![app-running-N2.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cmtMtOIaEpImIl0W8UlFPQ/app-running-N2.png)

4.  Stop the server before proceeding further by pressing `CTRL + C`.
    
5.  Create a ClusterIP service for exposing the application to the internet:
    

```
kubectl expose deployment/myapp

```

![explose-deployment.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/a1OUGm0MZ2-46JSV6JSV0g/explose-deployment.png)

::page{title="Exercise 2: Implement Vertical Pod Autoscaler (VPA)"}

Vertical Pod Autoscaler (VPA) helps you manage resource requests and limits for containers running in a pod. It ensures pods have the appropriate resources to operate efficiently by automatically adjusting the CPU and memory requests and limits based on the observed resource usage.

### Step 1: Create a VPA configuration

You will create a Vertical Pod Autoscaler (VPA) configuration to automatically adjust the resource requests and limits for the `myapp` deployment.

Explore the `vpa.yaml` file, which has the following content:

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: myvpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: myapp
  updatePolicy:
    updateMode: "Auto"  # VPA will automatically update the resource requests and limits

```

### Explanation

This YAML file defines a VPA configuration for the `myapp` deployment. The `updateMode: "Auto"` setting means that VPA will automatically update the resource requests and limits for the pods in this deployment based on the observed usage.

### Step 2: Apply the VPA

Apply the VPA configuration using the following command:

```
kubectl apply -f vpa.yaml

```

![apply-VPA.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/dMj1wayYuCmG9H-mMc6xtw/apply-VPA.png)

### Step 3: Retrieve the details of the VPA

1.  Retrieve the created VPA:
    

```
kubectl get vpa

```

![get-VPA-01.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/VDf0GEOnyTKVy16S8fKsgw/get-VPA-01.png)

This output shows that:

-   The VPA named `myvpa` is in Auto mode, recommending 25 milli-cores of CPU and 256 MB of memory for the pods it manages.
    
-   It has been created 29 seconds ago and has been providing these recommendations since then.
    

2.  Retrieve the details and current running status of the VPA.
    

```
kubectl describe vpa myvpa

```

![describe-VPA-name.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/TEnLkU5zlkxYKlP2qbgdfw/describe-VPA-name.png)

### Explanation

The output of `kubectl describe vpa myvpa` is providing recommendations for CPU and memory:

| Resource | Definition |
| --- | --- |
| Lower Bound | Minimum resources the VPA recommends. |
| Target | Optimal resources the VPA recommends. |
| Uncapped Target | Target without any predefined limits. |
| Upper Bound | Maximum resources the VPA recommends. |

| Resource | CPU | Memory |
| --- | --- | --- |
| Lower Bound | 25m | 256MiB (262144KiB) |
| Target | 25m | 256MiB |
| Uncapped Target | 25m | 256MiB |
| Upper Bound | 671m | 1.34GiB (1438074878KiB) |

###### These recommendations indicate that the VPA is functioning correctly and is providing target values based on observed usage.

::page{title="Exercise 3: Implement Horizontal Pod Autoscaler (HPA)"}

Horizontal Pod Autoscaler (HPA) automatically scales the number of pod replicas based on observed CPU/memory utilization or other custom metrics. VPA adjusts the resource requests and limits for individual pods. However, HPA changes the number of pod replicas to handle the load.

### Step 1: Create an HPA configuration

You will configure a Horizontal Pod Autoscaler (HPA) to scale the number of replicas of the `myapp` deployment based on CPU utilization.

Explore the `hpa.yaml` file, which has the following content:

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: myhpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 1         # Minimum number of replicas
  maxReplicas: 10        # Maximum number of replicas
  targetCPUUtilizationPercentage: 5  # Target CPU utilization for scaling

```

### Explanation

This YAML file defines a Horizontal Pod Autoscaler configuration for the `myapp` deployment. The HPA will ensure that the average CPU utilization across all pods remains close to 5%. If the utilization is higher, HPA will increase the number of replicas, and if it's lower, it will decrease the number of replicas within the specified range of 1 to 10 replicas.

### Step 2: Configure the HPA

Apply the HPA configuration:

```
kubectl apply -f hpa.yaml

```

![apply-HPA-N2.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/G9NUe23K6mQUPiDZgSuqEQ/apply-HPA-N2.png)

### Step 3: Verify the HPA

Obtain the status of the created HPA resource by executing the following command:

```
kubectl get hpa myhpa

```

![HPA-before-load-genrn.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/39VX--b47HRvR5AxPnp-rw/HPA-before-load-genrn.png)

This command provides details about the current and target CPU utilization and the number of replicas.

### Step 4: Start the Kubernetes proxy

Open another terminal and start the Kubernetes proxy:

```
kubectl proxy

```

![proxy-server'.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/fqnz73a29FVRNEHFTY7kiQ/proxy-server-.png)

### Step 5: Spam and increase the load on the app

Open another new terminal and enter the below command to spam the app with multiple requests for increasing the load:

```
for i in `seq 100000`; do curl -L localhost:8001/api/v1/namespaces/sn-labs-$USERNAME/services/myapp/proxy; done

```

![spamming-output.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/WZJmycDO1lnY1u-VQFX_Mw/spamming-output.png)  

**Proceed with further commands in the new terminal.**

### Step 6: Observe the effect of autoscaling

1.  Run the following command to observe the replicas increase in accordance with the autoscaling:
    

```
kubectl get hpa myhpa --watch

```

![autoscaling-RESULT.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/rlGAcjISF44jgmA3-5mdFg/autoscaling-RESULT.png)

2.  You will see an increase in the number of replicas, which shows that your application has been autoscaled.
    
3.  Terminate this command by pressing `CTRL + C`.
    

### Step 7: Observe the details of the HPA

1.  Run the following command to observe the details of the horizontal pod autoscaler:
    

```
kubectl get hpa myhpa

```

![HPA-AFTER-load-genrn.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/y4FaE9Ekdw_IJ39aHhC1Lg/HPA-AFTER-load-genrn.png)

2.  You will notice that the number of replicas has increased now.
    
3.  Stop the proxy and the load generation commands running in the other two terminals by pressing `CTRL + C`.
    

::page{title="Exercise 4: Create a Secret and update the deployment"}

Kubernetes Secrets lets you securely store and manage sensitive information, such as passwords, OAuth tokens, and SSH keys. Secrets are base64-encoded and can be used in your applications as environment variables or mounted as files.

### Step 1: Create a Secret

Explore the content of the file `secret.yaml`:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secret
type: Opaque
data:
  username: bXl1c2VybmFtZQ==
  password: bXlwYXNzd29yZA==

```

### Explanation

This YAML file defines a secret named `myapp-secret` with two key-value pairs: `username` and `password`. The values are base64-encoded strings.

### Step 2: Update the deployment to utilize the secret

Add the following lines at the end of `deployment.yaml`:

```
        env:
        - name: MYAPP_USERNAME
          valueFrom:
            secretKeyRef:
              name: myapp-secret
              key: username
        - name: MYAPP_PASSWORD
          valueFrom:
            secretKeyRef:
              name: myapp-secret
              key: password

```

### Explanation

-   `name:` - Defines the environment variables: 'MYAPP\_USERNAME' and 'MYAPP\_PASSWORD', respectively.
    
-   `valueFrom:` - Specifies that the value of the environment variable should be sourced from another location rather than being hardcoded.
    
-   `secretKeyRef:` - Indicates that the value of the environment variable should come from a Kubernetes secret.
    
-   `name: myapp-secret` - Specifies the name of the secret 'myapp-secret', from which to retrieve the value.
    
-   `key:` - Specifies which key within the secret is to be used for the value of the 'MYAPP\_USERNAME' and 'MYAPP\_PASSWORD' environment variables, respectively.
    

With these updates, the `myapp` application can now read these environment variables to get the required credentials, making it more secure and flexible.

### Step 3: Apply the secret and deployment

1.  Apply the secret using the following command:
    

```
kubectl apply -f secret.yaml

```

![apply-secret-N2.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/dOva6Sx_C5OtN9xmhmkwxw/apply-secret-N2.png)

2.  Apply the updated deployment using the following command:
    

```
kubectl apply -f deployment.yaml

```

![secret-dply-apply-N2.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/hMF2wnp4TFu3xGkRMaC2bA/secret-dply-apply-N2.png)

### Step 4: Verify the secret and deployment

You will now verify if the secret and the deployment using it have been applied.

1.  Run the following command to retrieve the details of `myapp-secret` showing its name, type, and creation timestamp:
    

```
kubectl get secret

```

![get-secret.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/e7z4D1DIfjQXKl32ncxfAw/get-secret.png)

2.  Run the following command to show the status of the deployment, including information about replicas and available replicas.
    

```
kubectl get deployment

```

![Ex3-secrets-get-deplymn.png](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/gkfpoSdpE2GnHDL_WetrtA/Ex3-secrets-get-deplymn.png)

::page{title="Conclusion"}

In this lab, you began by building and deploying an application called `myapp` on Kubernetes. Following this, you configured a Vertical Pod Autoscaler (VPA) to automatically adjust resource requests and limits for the `myapp` deployment. Subsequently, you implemented a Horizontal Pod Autoscaler (HPA) to scale the number of replicas for the `myapp` deployment based on CPU utilization. Finally, you created a Secret and updated the `myapp` deployment to utilize it.

## Author(s)

[Nikesh Kumar](https://www.linkedin.com/in/raonikeshnk)

### © IBM Corporation. All rights reserved.

&nbsp;
