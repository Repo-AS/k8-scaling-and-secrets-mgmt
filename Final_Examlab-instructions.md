::page{title="Lab (Option A: Python): Build and Deploy a Simple Guestbook App"}

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/images/IDSN-logo.png" width="200" alt="cognitiveclass.ai logo">

##

## Objectives

In this lab, you will:

- Build and deploy a simple Guestbook application
- Autoscale the Guestbook application using Horizontal Pod Autoscaler
- Perform Rolling Updates and Rollbacks

::page{title="Project Overview"}

## Guestbook application

Guestbook is a simple web application that we will build and deploy with Docker and Kubernetes. The application consists of a web front end, which will have a text input where you can enter any text and submit. For all of these, we will create Kubernetes Deployments and Pods. Then, we will apply Horizontal Pod Scaling to the Guestbook application and finally work on Rolling Updates and Rollbacks.

::page{title="Verify the environment and command line tools"}

1. If a terminal is not already open, open a terminal window by using the menu in the editor: `Terminal > New Terminal`.

> **Note:** Please wait for some time for the terminal prompt to appear.

![](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/images/Screenshot1.jpg) <br>

2. Change to your project folder.

>> **Note: If you are already on the `/home/project` folder, please skip this step.**

```
cd /home/project
```

3. Clone the git repository that contains the artifacts needed for this lab.

```
[ ! -d 'guestbook' ] && git clone https://github.com/ibm-developer-skills-network/guestbook
```

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/labs/5_FinalProject_Coursera/images/new_clone_command.png"> <br>

4. Change to the directory for this lab.

```
cd guestbook
```

5. List the contents of this directory to see the artifacts for this lab.

```
ls
```

::page{title="Creating the Guestbook app"}

# Note:

1. Please ensure that all updates to the files are properly saved.  

2. Download your project files and capture screenshots as mentioned in the respective tasks.  

3. As mentioned in the *\"Final Project Overview and Review Criteria\"*, you can submit your project deliverables through either Option 1: AI-Graded Submission and Evaluation or Option 2: Peer-Graded Submission and Evaluation.

4. **For Option 1: AI-Graded Submission and Evaluation:**  
   - Submission requires the terminal output or code snippet.

5. **For Option 2: Peer-Graded Submission and Evaluation:**  
   - Submission requires screenshots for **Tasks 1 to 10**  
## Build the guestbook app

To begin, we will build and deploy the web front end for the guestbook app.

1. Change to the `v1/guestbook` directory.

```
cd v1/guestbook
```

2. Dockerfile incorporates a more advanced strategy called multi-stage builds, so feel free to read more about that [here](https://docs.docker.com/develop/develop-images/multistage-build/).

Complete the Dockerfile with the necessary Docker commands to build and push your image. The path to this file is `guestbook/v1/guestbook/Dockerfile`.

<details>
<summary>Hint!</summary>
The FROM instruction initializes a new build stage and specifies the base image that subsequent instructions will build upon.<br>
The COPY command enables us to copy files to our image. <br>
The ADD command is used to copy files/directories into a Docker image. <br>
The RUN instruction executes commands.<br>
The EXPOSE instruction exposes a particular port with a specified protocol inside a Docker Container.<br>
The CMD instruction provides a default for executing a container, or in other words, an executable that should run in your container.<br>
</details>

**Assessment:**

**For Option 1: AI-Graded Submission and Evaluation:** Download  or copy and paste the Dockerfile and save it in a text file named `Dockerfile` for the final project submission and evaluation.

**For Option 2: Peer-Graded Submission and Evaluation:** Take a screenshot of the completed dockerfile and save it as `Dockerfile.jpeg` or `Dockerfile.png` for the Peer Assignment.

3. Export your namespace as an environment variable so that it can be used in subsequent commands.

```
export MY_NAMESPACE=sn-labs-$USERNAME
```

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/labs/5_FinalProject_Coursera/images/build_guestbook_3.png"> <br>

4. Build the guestbook app using the Docker Build command.

<details>

<summary>Hint!</summary>

```
docker build . -t us.icr.io/$MY_NAMESPACE/guestbook:v1
```

<br>
<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/labs/5_FinalProject_Coursera/images/build_guestbook_4.png"> <br>

</details>

5. Push the image to IBM Cloud Container Registry.

<details>

<summary>Hint!</summary>

```
docker push us.icr.io/$MY_NAMESPACE/guestbook:v1
```

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/labs/5_FinalProject_Coursera/images/build_guestbook_5.png"> <br>

</details>

> **Note:** If you have tried this lab earlier, there might be a possibility that the previous session is still persistent. In such a case, you will see a **'Layer already Exists'** message instead of the **\'Pushed\'** message  in the above output. We recommend you to proceed with the next steps of the lab.

6. Verify that the image was pushed successfully.

```
ibmcloud cr images
```

![](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/images/Screenshot6.jpg) <br>

**Assessment:**

**For Option 1: AI-Graded Submission and Evaluation:** Copy and paste the terminal output and save it in a text file named `crimages` for the final project submission and evaluation.

**For Option 2: Peer-Graded Submission and Evaluation:** Take a screenshot of the output of Step 6 and save it as `crimages.jpeg` or `crimages.png` for the Peer Assignment.

7. Open the `deployment.yml` file in the `v1/guestbook` directory and view the code for the deployment of the application:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: guestbook
  labels:
    app: guestbook
spec:
  replicas: 1
  selector:
    matchLabels:
      app: guestbook
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: guestbook
    spec:
      containers:
      - image: us.icr.io/<your sn labs namespace>/guestbook:v1
        imagePullPolicy: Always
        name: guestbook
        ports:
        - containerPort: 3000
          name: http
        resources:
          limits:
            cpu: 50m
          requests:
            cpu: 20m
```

> **Note:** Replace `<your sn labs namespace>` with your SN labs namespace. To check your SN labs namespace, please run the command `ibmcloud cr namespaces`

- It should look as below:

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/labs/5_FinalProject_Coursera/images/adding-deployment.yml.png"> <br>

8. Apply the deployment using:

```
kubectl apply -f deployment.yml
```

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/labs/5_FinalProject_Coursera/images/deployment%20configuration--after%20updation.png"> <br>

9. Open a New Terminal and enter the below command to view your application:

```
kubectl port-forward deployment.apps/guestbook 3000:3000
```

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/labs/5_FinalProject_Coursera/images/port-forward__inital-guestbook.jpg"> <br>

10. Launch your application on port 3000. Click on the Skills Network button on the right, it will open the **\"Skills Network Toolbox.\"** Then, click the **Other** then **Launch Application\.** From there, you should be able to enter the port and launch.

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/labs/5_FinalProject_Coursera/images/Launch_Application.PNG"> <br>

11. Now you should be able to see your running application. Please copy the app URL which will be given as below:

![](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/images/Screenshot11.jpg) <br>

**Assessment:**

**For Option 1: AI-Graded Submission and Evaluation:** Copy and paste the code of the file index.html and save it in a text file named `app` for the final project submission and evaluation. You will be required to provide the code from this file, showing the default title and header.

**For Option 2: Peer-Graded Submission and Evaluation:** Take a screenshot of your deployed application and save it as `app.jpeg` or `app.png` for the Peer Assignment.

12. Try out the guestbook by putting in a few entries. You should see them appear above the input box after you hit **Submit**.

::page{title="Autoscale the Guestbook application using Horizontal Pod Autoscaler"}

1. Autoscale the Guestbook deployment using `kubectl autoscale deployment`.

<details>

<summary>Hint!</summary>

```
kubectl autoscale deployment guestbook --cpu-percent=5 --min=1 --max=10
```

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/labs/5_FinalProject_Coursera/images/autoscale1.jpg"> <br>

</details>

2. You can check the current status of the newly-made HorizontalPodAutoscaler, by running:

```
kubectl get hpa guestbook
```

The current replicas is 0 as there is no load on the server.

**Assessment:**

**For Option 1: AI-Graded Submission and Evaluation:** Copy and paste the  terminal output and save it in a text file named `hpa` for the final project submission and evaluation.

**For Option 2: Peer-Graded Submission and Evaluation:** Take a screenshot of your Horizontal Pod Autoscaler and save it as `hpa.jpeg` or `hpa.png` for the Peer Assignment.

3. Open another new terminal and enter the below command to generate load on the app to observe the autoscaling. (Please ensure your port-forward command is running. In case you have stopped your application, please run the port-forward command to re-run the application at port 3000.)

```
kubectl run -i --tty load-generator --rm --image=busybox:1.36.0 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- <your app URL>; done"
```

- Please replace your app URL in the `<your app URL>` part of the above command.

> Note: Use the same copied URL which you obtained in step 11 of the previous task.

The command will be as below:

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/labs/5_FinalProject_Coursera/images/app-load-generation.jpg"> <br>

> **Note:** In case you get a `Load generator already exists` error, please suffix a number after `load-generator`, for example, `load-generator-1`, `load-generator-2`.

- You will keep getting an output similar as below which will indicate the increasing load on the app:

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/labs/5_FinalProject_Coursera/images/load-generation-2.png"> <br>

> **Note:** Continue further commands in the 1st terminal.

4. Run the below command to observe the replicas increase in accordance with the autoscaling:

```
kubectl get hpa guestbook --watch
```

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/labs/5_FinalProject_Coursera/images/hpa-watch-1.png"> <br>

5. Run the above command again after 5-10 minutes and you will see an increase in the number of replicas which shows that your application has been autoscaled.

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/labs/5_FinalProject_Coursera/images/hpa-watch__increased-RS.jpg"> <br>

**Assessment:**

**For Option 1: AI-Graded Submission and Evaluation:** Copy and paste the  terminal output of your Autoscaler details and save it in a text file named `hpa2` for the final project submission and evaluation.

**For Option 2: Peer-Graded Submission and Evaluation:** Take a screenshot of your Autoscaler details and save it as `hpa2.jpeg` or `hpa2.png` for the Peer Assignment.

6. Run the below command to observe the details of the horizontal pod autoscaler:

```
kubectl get hpa guestbook
```

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/labs/5_FinalProject_Coursera/images/get-hpa-final.png"> <br>

- Please close the other terminals where load generator and port-forward commands are running.

::page{title="Perform Rolling Updates and Rollbacks on the Guestbook application"}

> **Note:** Please run all the commands in the 1st terminal unless mentioned to use a new terminal.

1. Please update both the title and header in `index.html` to **Guestbook – v2**.

<details>
<summary>Hint!</summary>

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/labs/5_FinalProject_Coursera/images/guestbook-content-updation.png"> <br>

</details>

2. Run the below command to build and push your updated app image:

<details>
<summary>Hint!</summary>

```
docker build . -t us.icr.io/$MY_NAMESPACE/guestbook:v1 && docker push us.icr.io/$MY_NAMESPACE/guestbook:v1
```

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/MlsFsnB1LKNwMy3rL-gKRA/Task%206%20upguestbook-png.png"> <br>

</details>

**Assessment:**

**For Option 1: AI-Graded Submission and Evaluation:** Copy and paste the terminal output and save it in a text file named `upguestbook` for the final project submission and evaluation.Your saved output must include the final digest line shown after a successful push.

**For Option 2: Peer-Graded Submission and Evaluation:** Take a screenshot of your updated image  and save it as `upguestbook.jpeg` or `upguestbook.png` for the Peer Assignment.

3. Update the values of the CPU in the `deployment.yml` to **cpu: 5m** and **cpu: 2m** as below:

<details>
<summary>Hint!</summary>

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/labs/5_FinalProject_Coursera/images/deployment.yml__cpu-updation.png"> <br>

</details>

4. Apply the changes to the `deployment.yml` file.

<details>
<summary>Hint!</summary>

```
kubectl apply -f deployment.yml
```
	
<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/labs/5_FinalProject_Coursera/images/deployment%20configuration--after%20updation.png"> <br>

</details>

**Assessment:**

**For Option 1: AI-Graded Submission and Evaluation:** Copy and paste the  terminal output of Step 4 and save it in a text file named `deployment` for the final project submission and evaluation.

**For Option 2: Peer-Graded Submission and Evaluation:** Take a screenshot of the details of the output of Step 4 and save it as `deployment.jpeg` or `deployment.png` for the Peer Assignment.

5. Run the port-forward command again to start the app:

```
kubectl port-forward deployment.apps/guestbook 3000:3000
```

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/labs/5_FinalProject_Coursera/images/port-forward__inital-guestbook.jpg"> <br>

6. Launch your application on port 3000. Click on the Skills Network button on the right, it will open the **\"Skills Network Toolbox.\"** Then, click the **Other** then **Launch Application**. From there you should be able to enter the port and launch.

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/labs/5_FinalProject_Coursera/images/Launch_Application.PNG"> <br>

7. You will notice the updated app content as below:

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/labs/5_FinalProject_Coursera/images/updated_guestbook--browser.png"> <br>

**Assessment:**

**For Option 1: AI-Graded Submission and Evaluation:** Copy and paste the code of your updated index.html and save it in a text file named `up-app` for the final project submission and evaluation. You\'ll be asked to paste the updated code from index.html showing the title and header as \"Guestbook – v2\".

**For Option 2: Peer-Graded Submission and Evaluation:** Take a screenshot of your updated application and save it as `up-app.jpeg` or `up-app.png` for the Peer Assignment.
  > **Note:** Please stop the application before running the next steps.

8. Run the below command to see the history of deployment rollouts:
> **Note:** Before proceeding, please run the following commands to free up the namespace quota and avoid `FailedCreate: exceeded quota` errors during the rollback:
>
> ```
> kubectl delete hpa guestbook
> kubectl scale deployment guestbook --replicas=1
> ```
>Wait about 60 seconds for the extra pods to terminate before continuing.
```
kubectl rollout history deployment/guestbook
```

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/labs/5_FinalProject_Coursera/images/rollout-history_1.jpg"> <br>

9. Run the below command to see the details of Revision of the deployment rollout:

```
kubectl rollout history deployments guestbook --revision=2
```

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/labs/5_FinalProject_Coursera/images/rollout-history--rev2.jpg"> <br>

**Assessment:**

**For Option 1: AI-Graded Submission and Evaluation:** Copy and paste the  terminal output of the details of the correct Revision and save it in a text file named `rev` for the final project submission and evaluation.

**For Option 2: Peer-Graded Submission and Evaluation:** Take a screenshot of the details of the correct Revision and save it as `rev.jpeg` or `rev.png` for the Peer Assignment.

10. Run the below command to get the replica sets and observe the deployment which is being used now:

```
kubectl get rs
```

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/labs/5_FinalProject_Coursera/images/get_replicaset__before-undo-rollout.jpg"> <br>

11. Run the below command to undo the deployment and set it to Revision 1:

```
kubectl rollout undo deployment/guestbook --to-revision=1
```

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/labs/5_FinalProject_Coursera/images/undo%20rollout.png"> <br>

12. Run the below command to get the replica sets after the Rollout has been undone. The deployment being used would have changed as below:
 	
```
kubectl get rs
```

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/cc201/labs/5_FinalProject_Coursera/images/get%20replicaset--after%20undo%20rollout.png"> <br>

**Assessment:**

**For Option 1: AI-Graded Submission and Evaluation:** Copy and paste the  terminal output and save it in a text file named `rs` for the final project submission and evaluation.

**For Option 2: Peer-Graded Submission and Evaluation:** Take a screenshot of the output of Step 12 and save it as `rs.jpeg` or `rs.png` for the Peer Assignment.

Note: The rollout may take a few minutes to stabilize. Please wait until the replica set matches the state shown in the screenshot above before copying the output for future reference.
	
Congratulations! You have completed the final project for this course. Do not log out of the lab environment (you can close the browser though) or delete any of the artifacts created during the lab, as these will be needed for the next lab,`Optional: Deploy Guestbook App from the OpenShift Internal Registry`.

## Author(s)
Manvi Gupta

<!--
## Changelog
| Date | Version | Changed by | Change Description |
|------|--------|--------|---------|
| 2026-01-27 | 0.3 | Ritika | Command updated kubectl autoscale deployment|
| 2025-11-13 | 0.2 | Manvi Gupta | Lab updated for Mark and Peer review |
| 2025-11-12 | 0.1 | Manvi Gupta | Lab created and updated for Mark and Peer review |

## <h3 align="center"> © IBM Corporation 2020. All rights reserved. <h3/>
-->

## <h3 align="center"> © IBM Corporation. All rights reserved. <h3/>

