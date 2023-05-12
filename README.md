# Deploy MQ on CP4I using Jenkins
Jenkins is a very popular CI/CD tool that allows you to provision applications. In this tutorial, I will show you how to deploy an MQ queue manager, which its configurations to an Red Hat Openshift (using the MQ operators that comes with Cloud Pak for Integration).

Before you begin, you must have the following setup
- Red Hat OpenShift cluster installed (my version is 4.12.9)
- Cloud Pak for Integration installed on the OpenShift cluster (my version is 2022.4.1)
- Github account
- Clone this repository (https://github.com/blackt1g3r/mq-pipeline.git)

![image](https://github.com/blackt1g3r/mq-pipeline/assets/14035593/9e0395dc-73b5-490b-9e29-be4dde81f650)

# Setup Jenkins 
## Install Jenkins on Red Hat OpenShift

OpenShift comes with a built-in image for that. For our purposes, we’re going to create an ephemeral Jenkins server.

Under the **developer** tab, click the "**Add**" button and select “**All Services**”.

![image](https://github.com/blackt1g3r/mq-pipeline/assets/14035593/7b49a0cc-67ba-475f-8710-b6c0790ad415)

In the search tab, type "**jenkins ephemeral**".

![Screenshot 2023-05-12 at 09 27 08](https://github.com/blackt1g3r/mq-pipeline/assets/14035593/125ba879-9eb2-4424-9532-d3d7d652afb6)

Click  “**Instantiate Template**”

![Screenshot 2023-05-12 at 09 27 50](https://github.com/blackt1g3r/mq-pipeline/assets/14035593/43647e95-5b8b-4054-a961-747478e97f21)

and select **Create**.

![Screenshot 2023-05-12 at 09 28 38](https://github.com/blackt1g3r/mq-pipeline/assets/14035593/29507fc8-f893-4cc3-b625-758a361454e8)

This will create a Jenkins container and a url for us to access it.

Click the "**Topology**" tab to view the progress of the Jenkins server. Select the Jenkins deployment. When it has finished deploying, you will see one pod in the “running” state.

![image](https://github.com/blackt1g3r/mq-pipeline/assets/14035593/81910d00-d47c-4425-9154-9d72250b4582)

Next, we’ll verify that Jenkins deployed correctly. On that same application page, scroll down to the bottom and click the route url.

![Screenshot 2023-05-12 at 09 30 14](https://github.com/blackt1g3r/mq-pipeline/assets/14035593/b60adc80-93ba-40e7-938c-0d9acf87d2f2)

If you see the Jenkins login page, you've successfully created a Jenkins server!

## Create a credential in Jenkins to store IBM entitlement key
Login Jenkins by using openshift admin account

![Screenshot 2023-05-12 at 09 36 14](https://github.com/blackt1g3r/mq-pipeline/assets/14035593/ec49dfdf-c155-4678-ab9c-f9993b08b6a2)

Click on **Manage Jenkins** 

![Screenshot 2023-05-12 at 09 38 38](https://github.com/blackt1g3r/mq-pipeline/assets/14035593/568bee1a-3afb-4b45-b0e2-f23974215d13)

Click **Manage Credentials**

![Screenshot 2023-05-12 at 09 39 13](https://github.com/blackt1g3r/mq-pipeline/assets/14035593/f4eb5f08-447b-4d1b-aef2-9a0600f667e0)

Click on **Jenkins (global)** > **Global credentials (unrestricted)**

Click on **Add Credentials**, with the following setup.

kind: _Secret text_
Scope: _Global_
Secret: _(obtained from https://myibm.ibm.com/products-services/containerlibrary)
ID: _ibm_entitlement_key_

![image](https://github.com/blackt1g3r/mq-pipeline/assets/14035593/966b32ca-0e0f-48c5-8c13-8a7edac9fcc0)

This ID is reference by the Jenkins pipeline to retrieve the value of the **Secret**.

## Enable Jenkins service account to create objects in OpenShift (from terminal)

1. Run this command to find the service account in the namespace

oc get sa -n jenkins

The response should be:

![Screenshot 2023-05-12 at 12 44 08](https://github.com/blackt1g3r/mq-pipeline/assets/14035593/9e49e1cd-77d8-49cc-bfad-59639872155d)

2. Run this command to assign _cluster-admin_ role to the service account. I assigned _cluster-admin_ role, to simplify the setup; but you can create a custom role with the appropriates to do this.

![Screenshot 2023-05-12 at 12 44 33](https://github.com/blackt1g3r/mq-pipeline/assets/14035593/4b5fcf24-e025-4f98-83c8-cfe348bf43cb)


# Configure a pipeline to build MQ queue manager (from Jenkins Console)

1. Click on **Dashboard** > **New Item** > **Pipeline**. Provide a name (e.g. mq-pipeline) and click **Ok**.

2. Go to **Pipeline** section, specify the following and click **Save**.
- Definition: _Pipeline script from SCM_
- SCM: _Git_
- Repository URL: https://github.com/blackt1g3r/mq-pipeline.git
- Branch Specifier: */main
- Script Path: _Jenkinsfile_

![image](https://github.com/blackt1g3r/mq-pipeline/assets/14035593/db9f3799-7bac-49ca-8fb6-efbcf8ce4948)

# Perform test build (from Jenkins Console)

1. Click **Build Now**.
2. Click **Build History** > #1 > **Console Output**
3. When the pipeline is completed, you should see the following output, with the MQ Console URL, with login user and password. It also generates a ccdt.json to access the queue manager.

![Screenshot 2023-05-12 at 11 19 10](https://github.com/blackt1g3r/mq-pipeline/assets/14035593/cda82f2a-b55a-4a4f-8c43-826c7c6259d7)

4. You should see the status of the completed job (#3 as follows).
![image](https://github.com/blackt1g3r/mq-pipeline/assets/14035593/27eb3372-7793-440a-86ee-5859d27d7cd2)

5. Pipeline finished
![Screenshot 2023-05-12 at 11 42 03](https://github.com/blackt1g3r/mq-pipeline/assets/14035593/c9398397-ed9c-48d7-b103-b7117bfa62f7)

6. ccess MQ console:

![Screenshot 2023-05-12 at 11 44 29](https://github.com/blackt1g3r/mq-pipeline/assets/14035593/d6b9c380-c839-428b-b7a3-32e3d54e6b19)

# Test the deployed queue manager (from terminal)
1. Clone the git repository
git clone https://github.com/blackt1g3/mq-pipeline.git

2. Go to the folder mq-pipeline/test and update the file ccdt.json.

3. Put a message.

 ./mq-put.sh
 
![Screenshot 2023-05-12 at 11 35 19](https://github.com/blackt1g3r/mq-pipeline/assets/14035593/91eab5b8-e2a0-4132-b36c-5198bc9bd5ea)

4. Get the message

./mq-get.sh

![Screenshot 2023-05-12 at 11 36 13](https://github.com/blackt1g3r/mq-pipeline/assets/14035593/3d1148b1-5e3c-4ff6-8ee4-db7321fa479d)




