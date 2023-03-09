# Building and Deploying a Microservice Application on a Hybrid Cloud

# Prerequisites

* [AWS](https://aws.amazon.com/) account
* [Dockerhub](https://hub.docker.com/signup) account
* [GitHub](https://github.com/) account

# Create Python Environment

1. Create a [python virtual environment](https://docs.python.org/3/library/venv.html) and activate it
2. Setup the following modules using pip:

    * `pip install pybuilder`
    * `pip install setuptools wheel twine`
    * `Pip install Flask`

3. Create a pybuilder project by executing  the command `pyb --start-project`.
  
   Use the following values to create a basic project structure.

    * Project name: `hospitalService`
    * Use the `default` values for Source, Docs, Unittest and Scripts directories
    * Since we are not going to use **flake8** and **coverage** plugins, type `‘n’`. For **distutils** type `'y'`(this is important to successfully build the project)

# Building A Microservice Application

4. Refer to https://packaging.python.org/en/latest/guides/using-testpypi/ and register for a TestPyPI account. Once registered, generate an API token by following https://test.pypi.org/help/#apitoken

5. Now it's time to build and publish your application to Test Python Package Index. 

  To build the application run `pyb` from the root of your project. This generates the `target/dist` directory. Navigate to `target/dist/hospitalService-1.0.0/dist` to see the generated binary wheel and gzip’ed tar. 
  
  From here, execute `twine upload --repository testpypi *` to upload your distributions to TestPyPI using twine. Use the following when prompted for credentials,

    - username: `__token__ `
    - password: apitoken obtained in step 4

  You can see if your package has successfully uploaded by navigating to the URL https://test.pypi.org/project/hospitalService. Note that the hospitalService is the name of the project.

# Containerizing the Application

6. Let's use Docker to containerize your application. You can find a Dockerfile with related configurations in the root of the project. You may refer to https://docs.docker.com/engine/reference/builder/ to learn more about the syntax.

  Modify line 7 to download and install the application that we published to the PyPI index in previous step

7. Install Docker by referring to https://docs.docker.com/get-docker/ 

  Once installed, from the root of the project, execute `docker build -t hospital_service:1.0.0 .` to build and tag a docker image by reading the instructions from the Dockerfile.

8. Great job! We have built a flask based microservice and containerized it using Docker. Let's see how we can execute the service using the docker image that we created.

  To run the docker image, execute the following command `docker run -p 5000:5000 hospital_service:1.0.0`

  Once that is done, you should be able to access the service using any of following endpoints, `http://localhost:5000/doctors`, `http://localhost:5000/doctors/<speciality>`, `http://localhost:5000/hospitals/<hospital>`, where you need to replace the speciality and hospital with respective values

# Deploy the Containerized Application on AWS EKS

9. Now it's time to deploy the containerized application on a Hybrid Cloud, AWS EKS. To begin with, we need to sign in to [AWS management console](https://aws.amazon.com/console/) to create an EKS cluster.

10. Once signed in, search for Elastic Kubernetes Service in the search bar and then click on `Add Cluster -> Create`
 ![alt text](images/cluster_01.png)

11. Name the cluster as `hospitalService`. Create a new cluster service role by referring to the [EKS user guide](https://docs.aws.amazon.com/eks/latest/userguide/service_IAM_role.html#create-service-role). Once done, select the defined role and click on next
![alt text](images/cluster_02.png)

12. Keep the `default` values for rest of the steps (Specify Networking, Configure Logging, Select add-ons, Configure selected add-ons setting) and click on `next`. At the final step (Review and Create) keep everything to `default` and click on `create`

It may take upto 10-15 minutes to create the cluster. Once done, the cluster status should be Active.

13. Before deploying the application, we need to add nodes to the cluster. Head over to `Compute` tab and click on `Add node group`
![alt text](images/node_01.png)

14. The name of the node group the cluster should be the same (hospitalService).

  Refer to the [Node IAM role user guide](https://docs.aws.amazon.com/eks/latest/userguide/create-node-role.html#create-worker-node-role) and create a role.

  Note: At step 5.e, make sure to select the box left of `AmazonEC2FullAccess` policy addition to `AmazonEKSWorkerNodePolicy` and `AmazonEC2ContainerRegistryReadOnly`. Skip the instruction for AmazonEKS_CNI_Policy and adding tags under step 6.c, it's optional.

16. Once the role is created, select the role in Configure node group page and click on next
![alt text](images/node_02.png)

17. In the next page, configure the values as shown below and click on next
![alt text](images/node_03.png)

18. Keep the default values for Specify networking step. Once that is done, click on create to add the node group

It may a few minutes to create the node group. Once created, the node group status should be Active.

18. Now it's time to connect to your AWS EKS cluster. To do this we will be using AWS CLI and Kubectl command. 

  * Install AWS CLI: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
  * Install Kubectl: https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html

19. Once installed, follow the instructions at https://docs.aws.amazon.com/powershell/latest/userguide/pstools-appendix-sign-up.html and create an access key ID and secret access key

20. From the command line execute `aws configure` and use the following values to configure AWS credentials for the CLI

  * AWS Access Key ID [None]: << your access key >>
  * AWS Secret Access Key [None]: << your secret access key >>
  * Default region name [None]: << look at query param region in the cluster url >>
    
      example: https://us-west-2.console.aws.amazon.com/eks/home?region=us-west-2#/clusters
  * Default output format [None]: << keep it to default, press enter >>

21. Once installed create a kubeconfig file for your cluster by following the steps 1-3 at https://aws.amazon.com/premiumsupport/knowledge-center/eks-cluster-connection/

  * cluster_name: hospitalService
  * region: << your aws region >>

20. Test your configuration by executing `kubectl get nodes`. Example output:
  ```
  NAME                                          STATUS   ROLES    AGE   VERSION
ip-172-31-28-202.us-west-2.compute.internal   Ready    <none>   40m   v1.24.10-eks-48e63af
```

21. It's time to deploy your application. Execute `kubectl apply -f deployment.yaml` followed by `kubectl apply -f services.yaml`

22. To access the exposed service, we need to add a security rule to allow inbound traffic to your cluster. 
To do this, go to your cluster info page in AWS Management console and then click on the `Networking` tab.
Then click on the Cluster security group. 
![alt text](images/sg_01.png)

23. It should take you to Security Groups page. Here, click on the Security group ID
![alt text](images/sg_02.png)

24. This should show the details of the security group. From here, click on `Edit Inbound Rules` to add a new security rule
![alt text](images/sg_03.png)

25. Click on `Add rule` and define a new rule to allow All traffic as shown below (Type: All traffic, Source: Anywhere) and when you are done click on `Save rules`
![alt text](images/sg_04.png)






