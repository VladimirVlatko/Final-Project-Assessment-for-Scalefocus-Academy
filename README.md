<h1 align="center">Final Project Assessment</h1>

<h2 align="center">Deploy a WordPress on Kubernetes (using MicroK8s) with Helm and automation with Jenkins.</h2>


In this project, I utilized Kubernetes and Helm to deploy a WordPress site. Both Kubernetes and Helm were already installed on the system. However, instead of using Minikube, I chose to use MicroK8s for bonus points. Bellow, I will explain each step of the process, from installing MicroK8s to deploying the WordPress site using automation with Jenkins and accessing it through the browser.

**1. Installing MixroK8s.** I installed MicroK8s using the official guide provided on their website, which can be found at https://microk8s.io/#install-microk8s. The guide provided step-by-step instructions on how to install MicroK8s on different operating systems, and I followed the instructions for Windows. In order to check if it is properly installed and running I ran the command `microk8s status --wait-ready`. 
<p align="center">
  <img src="https://github.com/VladimirVlatko/Final-Project-Assessment-for-Scalefocus-Academy/assets/70275697/ddd550ad-294e-4365-b6b9-1719221bfe42" alt="image">
</p>

Additionally, I ran the command `cd .kube` followed by `microk8s config > config` to enable the native Windows version of kubectl to be used directly on my command-line interface. This allows me to use the simplified command `kubectl` instead of `microk8s kubectl` for every Kubernetes-related command.

**2. Downloading the chart.** After installing MicroK8s, I downloaded the Helm chart for WordPress from the following GitHub repository: https://github.com/bitnami/charts/tree/main/bitnami/wordpress. 
- In the `values.yaml` file, I modified line 534 by changing the service type from `LoadBalancer` to `ClusterIP`. By changing this, the WordPress service will be exposed internally within the Kubernetes cluster. This means that it will be accessible from other services running within the cluster but not from outside the cluster. 
- Also, to meet another one of the prerequisites of the WordPress Helm chart, which requires PV provisioner support in the underlying infrastructure, I enabled the storage add-on in MicroK8s by running the command `microk8s enable storage`. This add-on ensures that the necessary infrastructure components are in place to dynamically provision and manage PersistentVolumes (PVs) and PersistentVolumeClaims (PVCs) for the WordPress deployment.

**3. Creating Jenkins Pipeline.** Next, I created a Jenkins pipeline called "WordPress Deployment" that checks if the `wp` namespace exists. If it doesn't exist, the pipeline creates the namespace. Additionally, the pipeline checks if WordPress is already deployed. If it is not deployed, the pipeline proceeds to install the WordPress Helm chart. In the end, the pipeline performs port forwarding to make the WordPress site accessible outside the cluster. This is how I configured the pipeline script: 

<p align="center">
  <img src="https://github.com/VladimirVlatko/Final-Project-Assessment-for-Scalefocus-Academy/assets/70275697/9f16e79c-9d3a-4f0f-8ed1-ad9e5ff96e88" alt="image">
</p>

***Here is an explanation of the pipeline components.***

***Agent***: The pipeline is configured to run on any available agent in the Jenkins environment.

***Stages***: The pipeline consists of three stages:

***a. Check and create wp namespace***: This stage checks if the "wp" namespace exists in the Kubernetes cluster. It does this by running the `kubectl get namespace command`. The result is captured in the `wpNamespace` variable. If the exit status of the command is not 0 (indicating that the namespace doesn't exist), the pipeline proceeds to create the "wp" namespace using the `kubectl create namespace` command. Otherwise, if the namespace already exists, the pipeline echoes a message indicating its existence.

***b. Check and install WordPress***: This stage checks if the WordPress Helm chart is installed in the "wp" namespace. It does this by running the `helm list command` with the `--kubeconfig` flag set to the specified path of the MicroK8s configuration file. The result is captured in the `wpDeployment` variable. If the exit status of the command is not 0 (indicating that the chart is not installed), the pipeline proceeds with the installation. It first builds the chart dependencies using `helm dependency build` for the specified chart directory. Then, it installs the WordPress Helm chart using the `helm install command` and the name of the deployment "final-project-wp-scalefocus". The command also includes, namespace "wp", values file path, and the `--kubeconfig` flag set to the MicroK8s configuration file path. If the chart is already installed, the pipeline echoes a message indicating its existence.

***c. Port Forwarding***: The "Port Forwarding" stage in the pipeline sets up port forwarding to access the deployed WordPress site. It waits for 1 minute to ensure the deployment is ready, and then runs the `kubectl port-forward --namespace wp svc/final-project-wp-scalefocus-wordpress 8081:80` command to forward traffic from local port 8081 to the WordPress service on port 80. This allows accessing the WordPress site via http://localhost:8081. Note that I used port 8081 instead of the default port 8080, as it was already in use by Jenkins.

**4. Deploying the WordPress Helm Chart using the Jenkins pipeline.** After configuring the pipeline, I initiated the build to deploy the WordPress Helm Chart. The initial stages completed quickly, and then the pipeline entered a 1-minute sleep before proceeding to the port forwarding stage. The port forwarding stage will continue to run until it is manually terminated or interrupted, enabling access to the WordPress site outside the cluster. The console output displayed the following information:

```Started by user Vladimir Velkovski
Started by user Vladimir Velkovski
[Pipeline] Start of Pipeline
[Pipeline] node
Running on Jenkins in C:\ProgramData\Jenkins\.jenkins\workspace\WordPress Deployment
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Check and create wp namespace)
[Pipeline] script
[Pipeline] {
[Pipeline] sh
+ kubectl get namespace wp --no-headers=true -o name
Error from server (NotFound): namespaces "wp" not found
[Pipeline] echo
Creating wp namespace
[Pipeline] sh
+ kubectl create namespace wp
namespace/wp created
[Pipeline] }
[Pipeline] // script
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Check and install WordPress)
[Pipeline] script
[Pipeline] {
[Pipeline] sh
+ helm list -n wp --short --kubeconfig C:/Windows/System32/config/systemprofile/AppData/Local/MicroK8s/config
+ grep final-project-wp-scalefocus
[Pipeline] echo
Installing WordPress Helm chart
[Pipeline] sh
+ helm dependency build 'C:/Users/V&M/Desktop/final/wordpress'
Saving 3 charts
Downloading memcached from repo oci://registry-1.docker.io/bitnamicharts
Pulled: registry-1.docker.io/bitnamicharts/memcached:6.4.2
Digest: sha256:ac800af4f9b6be921043eb5cd2ba07828ad6fc404b5762f2630657d9fdf5a6fe
Downloading mariadb from repo oci://registry-1.docker.io/bitnamicharts
Pulled: registry-1.docker.io/bitnamicharts/mariadb:12.2.2
Digest: sha256:f18fd0e930041ef6a1dff0789eb801f2c4c52f1e8e0ff7c610b109ae8304d74c
Downloading common from repo oci://registry-1.docker.io/bitnamicharts
Pulled: registry-1.docker.io/bitnamicharts/common:2.2.5
Digest: sha256:a088a039a53958fdd4ddff5a9799c0dba38d1c480bc768a9141cb87e7fcf7036
Deleting outdated charts
[Pipeline] sh
+ helm install final-project-wp-scalefocus 'C:/Users/V&M/Desktop/final/wordpress' -n wp -f 'C:/Users/V&M/Desktop/final/wordpress/values.yaml' --kubeconfig C:/Windows/System32/config/systemprofile/AppData/Local/MicroK8s/config
NAME: final-project-wp-scalefocus
LAST DEPLOYED: Tue May 16 12:53:12 2023
NAMESPACE: wp
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: wordpress
CHART VERSION: 16.1.2
APP VERSION: 6.2.0

** Please be patient while the chart is being deployed **

Your WordPress site can be accessed through the following DNS name from within your cluster:

    final-project-wp-scalefocus-wordpress.wp.svc.cluster.local (port 80)

To access your WordPress site from outside the cluster follow the steps below:

1. Get the WordPress URL by running these commands:

   kubectl port-forward --namespace wp svc/final-project-wp-scalefocus-wordpress 80:80 &
   echo "WordPress URL: http://127.0.0.1//"
   echo "WordPress Admin URL: http://127.0.0.1//admin"

2. Open a browser and access WordPress using the obtained URL.

3. Login with the following credentials below to see your blog:

  echo Username: user
  echo Password: $(kubectl get secret --namespace wp final-project-wp-scalefocus-wordpress -o jsonpath="{.data.wordpress-password}" | base64 -d)
[Pipeline] }
[Pipeline] // script
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Port Forwarding)
[Pipeline] script
[Pipeline] {
[Pipeline] sleep
Sleeping for 1 min 0 sec
[Pipeline] sh
+ kubectl port-forward --namespace wp svc/final-project-wp-scalefocus-wordpress 8081:80
Forwarding from 127.0.0.1:8081 -> 8080
Forwarding from [::1]:8081 -> 8080
```
Here is a screenshot of the build runtime in Jenkins.

<p align="center">
  <img src="https://github.com/VladimirVlatko/Final-Project-Assessment-for-Scalefocus-Academy/assets/70275697/94beb2cb-da62-4f8d-9678-e8da52e9beca" alt="image">
</p>


**5. Load the home page of the WordPress.** After successfully deploying the WordPress Chart and setting up port forwarding from local port 8081 to the WordPress service on port 80, I was able to access the WordPress site by visiting http://localhost:8081 in my web browser. Here is a screenshot of the homepage.

<p align="center">
  <img src="https://github.com/VladimirVlatko/Final-Project-Assessment-for-Scalefocus-Academy/assets/70275697/f4b271ce-449d-4a21-960d-f09543c71fb3" alt="image">
</p>

**6. Login to WordPress admin panel.** Additionally, I retrieved the password from the secret by using the command `kubectl get secret --namespace wp final-project-wp-scalefocus-wordpress -o jsonpath="{.data.wordpress-password}" | base64 --decode`. Then, I logged into the WordPress admin panel by accessing http://localhost:8081/wp-admin/ and entering the user name and the retrieved password.

<p align="center">
  <img src="https://github.com/VladimirVlatko/Final-Project-Assessment-for-Scalefocus-Academy/assets/70275697/c9f125cc-b907-43e2-87e3-defd815f7976" alt="image">
</p>

<p align="center">
  <img src="https://github.com/VladimirVlatko/Final-Project-Assessment-for-Scalefocus-Academy/assets/70275697/91007e04-eeb7-4d39-b5f9-83aaaf8483b1" alt="image">
</p>

**CONCLUSION.** In conclusion, through this project, I have gained valuable hands-on experience in deploying WordPress using Helm and Jenkins. This final exam project has allowed me to showcase my level of proficiency in setting up the necessary tools, making configuration changes, and automating the deployment process. It has been a rewarding journey of learning and applying DevOps principles, culminating in the successful deployment of a functional WordPress site. I am confident that the skills and knowledge acquired from this academy will serve as a solid foundation for my future endeavors in the field of DevOps.

