# Building and Deploying a Microservice Application on a Hybrid Cloud

## Objectives:

* Write a simple microservice application using Flask
* Containerize the microservice application using Docker
* Write a simple K8 deployment and expose it as a service
* Configure and deploy containerized application using Docker and K8 on AWS EKS
* Configure a CI/CD pipeline using GitHub Actions to automatically build and deploy code changes 

## Prerequisites

* [AWS](https://aws.amazon.com/) account
* [Dockerhub](https://hub.docker.com/signup) account
* [GitHub](https://github.com/) account

# Create Python Environment and Write a Microservice

1. Create a [python virtual environment](https://docs.python.org/3/library/venv.html) and activate it
2. Setup the following modules using pip:

    * `pip install pybuilder`
    * `pip install setuptools wheel twine`
    * `Pip install Flask`

3. Extract the hospitalService.zip
4. Open `hospitalService.py` file (located in src/main/python/hospitalService.py) and write a function to return details of doctors grouped by hospital's name.
The path should look like `/hospital/<hospitalName>`
5. Once completed, execute `flask --app hospitalService run` from the directory `src/main/python/` to deploy the service on a development server
Example output:
```bash
src/main/python ➜ flask --app hospitalService run
 * Serving Flask app 'hospitalService'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on http://127.0.0.1:5000
Press CTRL+C to quit
```
6. Use curl or postman and send a GET request to `http://127.0.0.1:5000/doctors` (You can also simply go to the URL in a browser). 
You should see the following output,
```json
[
    {
        "hospital": "Green Country Community Hospital",
        "id": 1,
        "name": "Jeremy Willey",
        "speciality": "Surgery"
    },
    {
        "hospital": "Angelwing Clinic",
        "id": 2,
        "name": "Camille Modin",
        "speciality": "Gynaecology"
    },
    {
        "hospital": "Little River Hospital",
        "id": 3,
        "name": "Pearl Cruce",
        "speciality": "Paediatric"
    },
    {
        "hospital": "Rosemary Hospital Center",
        "id": 4,
        "name": "Don Maccament",
        "speciality": "Surgery"
    },
    {
        "hospital": "Rosemary Hospital Center",
        "id": 5,
        "name": "Andrew Tessmer",
        "speciality": "Gynaecology"
    },
    {
        "hospital": "Angelwing Clinic",
        "id": 6,
        "name": "Barrett Cannata",
        "speciality": "Ophthalmology"
    }
]
```
Similarly, you can try sending requests to other routes:
http://127.0.0.1:5000/doctors/<speciality>, 
http://127.0.0.1:5000/hospitals/<hospital> where you need to replace <speciality> and <hospital> with respective values.

7. Makesure to stop the deployment server (CTRL+C) before moving on to the next section.

# Building and Publishing the Application

8. In this section, we are going to build and publish your microservice to Python Package Index.

Refer to https://packaging.python.org/en/latest/guides/using-testpypi/ and register for a TestPyPI account. Once registered, generate an API token by following https://test.pypi.org/help/#apitoken

9. Now it's time to build and publish your application to Test Python Package Index. 
    
To publish projects to PyPI repository, the project name must be unique. Refer to https://test.pypi.org/help/#project-name for more details.
Therefore, modify line 9 in `build.py` file to have a unique name (e.g: hospitalService_<your_name>)

To build the application run `pyb` from the root of your project. This generates the `target/dist` directory. Navigate to `target/dist/hospitalService_<your_name>-1.0.0/dist` to see the generated binary wheel and gzip’ed tar. 
  
  From here, execute `twine upload --repository testpypi *` to upload your distributions to TestPyPI using twine. Use the following when prompted for credentials,

    - username: `__token__ `
    - password: apitoken obtained in step 4

  You can see if your package has been successfully uploaded by navigating to the URL https://test.pypi.org/project/hospitalService-<your_name>/

# Containerizing the Application

10. Let's use Docker to containerize your application. You can find a Dockerfile with related configurations in the root of the project. You may refer to https://docs.docker.com/engine/reference/builder/ to learn more about the syntax.

Modify line 8 to download and install the application that we published to the PyPI index in previous step (hint: use pip)

11. Install Docker by referring to https://docs.docker.com/get-docker/ 

  Once installed, from the root of the project, execute `docker build -t <docker_username>/hospital_service:1.0.0 .` to build and tag a docker image by reading the instructions from the Dockerfile
  (replace `docker_username` with your actual docker username).

12. Great job! We have built a flask based microservice and containerized it using Docker. Let's see how we can execute the service using the docker image that we created.

  To run the docker image, execute the following command `docker run -p 5000:5000 <docker_username>/hospital_service:1.0.0`

  Once that is done, you should be able to access the service using any of following endpoints, `http://localhost:5000/doctors`, `http://localhost:5000/doctors/<speciality>`, `http://localhost:5000/hospitals/<hospital>`, where you need to replace the speciality and hospital with respective values

13. Press CTRL+C from the command line to stop the docker container

14. Before moving on to the next section, let's see how to host docker images.

[Docker Hub](https://hub.docker.com/) is a hosted repository service provided by Docker for finding and sharing container images.
First run `docker login` from command line and provide your docker credentials. 

Then run `docker push <docker_username>/hospital_service:1.0.0` to publish your hospital_service image to docker hub.

# Deploy the Containerized Application on AWS EKS

15. Now it's time to deploy the containerized application on a Hybrid Cloud, AWS EKS. To begin with, we need to sign in to [AWS management console](https://aws.amazon.com/console/) to create an EKS cluster.

16. Once signed in, search for Elastic Kubernetes Service in the search bar and then click on `Add Cluster -> Create`
 ![alt text](images/cluster_01.png)

17. Name the cluster as `hospitalService`. Create a new cluster service role by referring to the [EKS user guide](https://docs.aws.amazon.com/eks/latest/userguide/service_IAM_role.html#create-service-role). Once done, select the defined role and click on next
![alt text](images/cluster_02.png)

18. Keep the `default` values for rest of the steps (Specify Networking, Configure Logging, Select add-ons, Configure selected add-ons setting) and click on `next`. At the final step (Review and Create) keep everything to `default` and click on `create`

It may take upto 10-15 minutes to create the cluster. Once done, the cluster status should be Active.

19. Before deploying the application, we need to add nodes to the cluster. Head over to `Compute` tab and click on `Add node group`
![alt text](images/node_01.png)

20. The name of the node group the cluster should be the same (hospitalService).

  Refer to the [Node IAM role user guide](https://docs.aws.amazon.com/eks/latest/userguide/create-node-role.html#create-worker-node-role) and create a role.

  Note: At step 5.e, make sure to select the box left of `AmazonEC2FullAccess` policy addition to `AmazonEKSWorkerNodePolicy` and `AmazonEC2ContainerRegistryReadOnly`. Skip the instruction for AmazonEKS_CNI_Policy and adding tags under step 6.c, it's optional.

21. Once the role is created, select the role in Configure node group page and click on next
![alt text](images/node_02.png)

22. In the next page, configure the values as shown below and click on next
![alt text](images/node_03.png)

23. Keep the default values for Specify networking step. Once that is done, click on create to add the node group

It may a few minutes to create the node group. Once created, the node group status should be Active.

24. Now it's time to connect to your AWS EKS cluster. To do this we will be using AWS CLI and Kubectl command. 

  * Install AWS CLI: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
  * Install Kubectl: https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html

25. Once installed, follow the instructions at https://docs.aws.amazon.com/powershell/latest/userguide/pstools-appendix-sign-up.html and create an access key ID and secret access key

26. From the command line execute `aws configure` and use the following values to configure AWS credentials for the CLI

  * AWS Access Key ID [None]: << your access key >>
  * AWS Secret Access Key [None]: << your secret access key >>
  * Default region name [None]: << look at query param `?region` in the cluster url >>
    
      example: https://us-west-2.console.aws.amazon.com/eks/home?region=us-west-2#/clusters, here the region is _us-west-2_
  * Default output format [None]: << keep it to default, press enter >>

27. Once installed create a kubeconfig file for your cluster by following the steps 1-3 at https://aws.amazon.com/premiumsupport/knowledge-center/eks-cluster-connection/

  * cluster_name: hospitalService
  * region: << your aws region >>

28. Test your configuration by executing `kubectl get nodes`. Example output:
  ```
  NAME                                          STATUS   ROLES    AGE   VERSION
ip-172-31-28-202.us-west-2.compute.internal   Ready    <none>   40m   v1.24.10-eks-48e63af
```

29. It's time to deploy your application.  
To give some K8s context, a [Pod](https://kubernetes.io/docs/concepts/workloads/pods/) is the most basic deployable unit within a Kubernetes cluster. A Pod runs one or more containers. 
A [Kubernetes Deployment](https://www.vmware.com/topics/glossary/content/kubernetes-deployment.html#:~:text=A%20Kubernetes%20Deployment%20tells%20Kubernetes,earlier%20deployment%20version%20if%20necessary.) tells Kubernetes how to create or modify instances of the pods that hold a containerized application.

Let's create a deployment to hold an instance of your containerized application. Open `deployment.yaml` file and modify line 18 to have your docker image (e.g.: <docker_username>/hospital_service:1.0.0)

Once that is done, run `kubectl apply -f deployment.yaml` from command line to create a deployment on our AWS EKS cluster.

A [Kubernetes Service](https://kubernetes.io/docs/concepts/services-networking/service/) is a method for exposing an application that is running as one or more Pods in your cluster.
We need to create a service to access the microservice from outside the cluster. Run `kubectl apply -f services.yaml` from command line to create a service exposing the deployment that we created in the above step

32. To access the exposed service from your computer, you need to add a security rule to allow inbound traffic to your cluster. 
To do this, go to your cluster info page in AWS Management console and then click on the `Networking` tab.
Then click on the Cluster security group. 
![alt text](images/sg_01.png)

33. It should take you to Security Groups page. Here, click on the Security group ID
![alt text](images/sg_02.png)

34. This should show the details of the security group. From here, click on `Edit Inbound Rules` to add a new security rule
![alt text](images/sg_03.png)

35. Click on `Add rule` and define a new rule to allow All traffic as shown below (Type: All traffic, Source: Anywhere) and when you are done click on `Save rules`
![alt text](images/sg_04.png)

36. A [CI/CD pipeline](https://about.gitlab.com/topics/ci-cd/) automates a series of steps that must be performed to deliver a software.
We are going to build such a pipeline to automatically deploy code changes. Once you make a change in `hospitalService.py`, 
the pipeline will take care of the rest of the steps that we did in the previous sections, from building the python application to deploying it on the AWS EKS.

There are many tools to build CI/CD pipelines. For this lab we will be using [GitHub Actions](https://github.com/features/actions).
To begin with, [Create a GitHub repository](https://docs.github.com/en/get-started/quickstart/create-a-repo) and [commit](https://docs.github.com/en/repositories/working-with-files/managing-files/adding-a-file-to-a-repository) all of your project files.





