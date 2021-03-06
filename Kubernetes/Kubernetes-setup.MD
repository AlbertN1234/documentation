# Setup Kubernetes (K8s) Cluster on AWS


1. Create Ubuntu EC2 instance
1. install AWSCLI
   ```sh
    curl https://s3.amazonaws.com/aws-cli/awscli-bundle.zip -o awscli-bundle.zip
    apt install unzip python
    unzip awscli-bundle.zip
    #sudo apt-get install unzip - if you dont have unzip in your system
    ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
    ```

1. Install kubectl on ubuntu instance
   ```sh
   curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
    chmod +x ./kubectl
    sudo mv ./kubectl /usr/local/bin/kubectl
   ```

1. Install kops on ubuntu instance
   ```sh
    curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
    chmod +x kops-linux-amd64
    sudo mv kops-linux-amd64 /usr/local/bin/kops
    ```
1. Create an IAM role with VPC full access, Route53 full access, EC2 full access, IAM full access and S3 full access

1. Attach IAM role to ubuntu instance
   ```sh
   # Note: If you create IAM user with programmatic access then provide Access keys. Otherwise region information is enough
   run this command and provide the details below:  aws configure
   AWS Access Key ID: (if you create a user instead of a role, you need to provide this info. If you create a role, just hit 'enter' on the keyboard)
   AWS Secret Access Key: (if you create a user instead of a role, you need to provide this info. If you create a role, just hit 'enter' on the keyboard)
   Default region name: (enter the region where your vpc is located or the region given to you)
   Default output format: (hit 'enter' keyboard)
   

    ```

1. Create a Route53 private hosted zone (you can create Public hosted zone if you have a domain)
   ```sh
   Route53 --> hosted zones --> created hosted zone  
   Domain Name: valaxy.net
   Type: Private hosted zone for Amzon VPC
   Region: the same region provided on the previous question
   VPC: you can use your default vpc or the vpc that will be provided to you
   ```

1. create an S3 bucket
   ```sh
    aws s3 mb s3://demo.k8s.valaxy.net
   ```
1. Expose environment variable:
   ```sh
    export KOPS_STATE_STORE=s3://demo.k8s.valaxy.net
   ```

1. Create sshkeys before creating cluster
   ```sh
    ssh-keygen
   ```

1. Create kubernetes cluster definitions on S3 bucket
   ```sh
   kops create cluster   --cloud=aws   --zones=ap-south-1b (availability zone of your instance) --name=demo.k8s.valaxy.net (name of the cluster) --dns-zone=valaxy.net (name of the dns)   --dns private 
   #remove all the description in the parenthesis when you want to run the command. They are only there for comments
    ```

1. Create kubernetes cluster
    ```sh
    kops update cluster demo.k8s.valaxy.net --yes
    ```

1. Wait untill the cluster is up running (check if master and 2 nodes got created on your EC2 dashboard) then Validate your cluster and login in it
     ```sh
      kops validate cluster   (to validate your cluster)
      ssh -i ~/.ssh/id_rsa ubuntu@api.demo.k8s.valaxy.net     (to login in to your master server)
      #remember you can either use the above command to login in your master or you can go on the regular way to login using Mobxterm)
    ```

1. To list nodes
   ```sh
   kubectl get nodes
   ```

1. To delete cluster
    ```sh
     kops delete cluster demo.k8s.valaxy.net --yes
    ```
   
#### Deploying Nginx pods on Kubernetes
1. Deploying Nginx Container
    ```sh
    kubectl run sample-nginx --image=nginx --replicas=2 --port=80
    # kubectl run simple-devops-project --image=yankils/simple-devops-image --replicas=2 --port=8080
    kubectl get pods
    kubectl get deployments
   ```

1. Expose the deployment as service. This will create an ELB in front of those 2 containers and allow us to publicly access them.
   ```sh
   kubectl expose deployment sample-nginx --port=80 --type=LoadBalancer
   # kubectl expose deployment simple-devops-project --port=8080 --type=LoadBalancer
   kubectl get services -o wide
   ```
