## Note
This README is out of date because it was written to release 1.1.


# How-to Deploy Orc8r/NMS AWS deployment
[![Build Status](https://img.shields.io/badge/made%20by-juliogaraujo-blue)](https://www.linkedin.com/in/araujojulio/) [![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://github.com/juliogaraujo/magma) [![Build Status](https://img.shields.io/badge/reference-Documentation%20Magma-red)](https://magma.github.io/magma/docs/basics/introduction.html)


## Note for remote deployment:
In order to save all logs and prevent any terminal disconnection it is strong recommend the use of the following commands:

#### A - Open a screen shell
<pre>screen</pre>
<pre> 
<p><b>OBS:</b> to recovery the terminal use:</p>
screen -list 
screen -r 
</pre>

#### B - Use the follow command in order to save all commands and results:
script -torc8r-XX.timing orc8r-XX.log<br>
<b>OBS:</b> XX = 00,01,02..N


## 1 - Prerequisites for linux Ubuntu 18.04 Server.
#### 1.1 - Fresh installation of Ubuntu Server 18.04. (For this case it was used the VirtualBox, but you can use VMWare or any other virtual machine tools that you might be familiarize of)
This fresh instalation of Ubuntu Server will be used to build the docker images and deploy them towards the docker hub.
Recommend required for this virtual machine.
- 2 vCPU
- 2048 Gb MEM
- 50 Gb HD


#### 1.2 - Install the latest docker:
<pre>
  $ sudo apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common </code>
  $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
  $ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
  $ sudo apt update
  $ sudo apt install docker-ce docker-ce-cli containerd.io
</pre>

Ref: https://docs.docker.com/engine/install/ubuntu/


#### 1.3 - Install the latest docker-compose
<pre>
  $ sudo curl -L "https://github.com/docker/compose/releases/download/1.26.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
  $ sudo chmod +x /usr/local/bin/docker-compose
</pre>
   
Ref: https://docs.docker.com/compose/install/




#### 1.4 - Install the folling tools:

Optional pyenv (Only for MacOS): https://github.com/pyenv/pyenv#installation
                    
For Ubuntu: https://gist.github.com/luzfcb/ef29561ff81e81e348ab7d6824e14404

##### Install the python3.7
<pre> 
  $ sudo apt install python3.7 
  $ sudo apt install python3-pip
  $ pip3 install ansible fabric3 jsonpickle requests PyYAML
</pre>
Ref: https://dev.to/serhatteker/how-to-upgrade-to-python-3-7-on-ubuntu-18-04-18-10-5hab

##### aws-iam-authenticator
<pre>      
  $ mkdir -p ~/Downloads/aws-iam-auth; cd ~/Downloads/aws-iam-auth
  $ curl -o aws-iam-authenticator https://amazon-eks.s3.us-west-2.amazonaws.com/1.17.9/2020-08-04/bin/linux/amd64/aws-iam-authenticator
  $ chmod a+x aws-iam-authenticator
  $ sudo cp aws-iam-authenticator /usr/local/bin
</pre>
Ref: https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html


##### kubectl (kubernetes-cli)
<pre>
  $ sudo snap install kubectl --classic
</pre>
       
##### helm (kubernetes-helm)
Download your desired version (https://github.com/helm/helm/releases)
<pre>
 $ mkdir -p ~/Downloads/helm/; cd ~/Downloads/helm/
 $ wget https://get.helm.sh/helm-v3.3.1-linux-amd64.tar.gz
 $ tar -xzvf helm-v3.3.1-linux-amd64.tar.gz
 $ sudo cp linux-amd64/helm /usr/local/bin/
</pre>   
Ref: https://helm.sh/docs/intro/install/     
<br>
Hosting helm private repository from Github, to be used in Item 7.   
Ref: https://blog.softwaremill.com/hosting-helm-private-repository-from-github-ff3fa940d0b7
        
        
##### terraform
<pre>
 $ mkdir -p ~/Downloads/terraform; cd ~/Downloads/terraform
 $ wget https://releases.hashicorp.com/terraform/0.13.2/terraform_0.13.2_linux_amd64.zip
 $ unzip terraform_0.13.2_linux_amd64.zip
 $ sudo cp terraform /usr/local/bin/
</pre>        
Ref: https://www.terraform.io/downloads.html

##### awscli
<pre>
 $ sudo apt install awscli
 $ aws configure
 AWS Access Key ID [None]: 
 AWS Secret Access Key [None]: 
 Default region name [None]: 
 Default output format [None]: 
</pre>           
Access key ID and secret access key 
Ref: https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html
             
##### boto3
<pre>
 $ sudo apt install python3-boto3
</pre>

#### 1.5 - Download Magma from git-hub
<pre>
 $ mkdir -p ~/Magma/; cd ~/Magma
 $ wget https://github.com/magma/magma/archive/v1.1.0.zip
 $ unzip v1.1.0.zip
</pre>

#### 2 - Run the build command:
<pre>
 $ cd ~/Magma/magma-1.1.0/orc8r/cloud/docker
 $ sudo su 
 $ ./build.py -a
     
 ...
 <b>Finished printout would be:</b>
    Successfully built 2fef953fe12e
    Successfully tagged orc8r_test:latest
</pre>

#### 3 - Create a docker hub account and upload the images to this account.
 hub.docker.com<br>
 Docker account: {dockerAccount}
 
#### 4 - Publish the images:
<pre>
  # docker logout
  # docker login
  {dockerAccount}
  {password}
  ...
  <b>Login Succeeded</b>
</pre>

<pre>
 # echo "{password}" > psw.txt
 # export MAGMA_TAG=v1.1.0-master
 # for image in proxy controller prometheus-cache alertmanager-configurer prometheus-configurer grafana
 > do
 >    ../../../orc8r/tools/docker/publish.sh -r docker.io/{dockerAccount} -i ${image} -v ${MAGMA_TAG} -u {dockerAccount} -p ./psw.txt
 > done
 ...
 <b>Login Succeeded</b>
</pre>	


> The push refers to repository [docker.io/{dockerAccount}/prometheus-configurer] <br/>
> 775392ca3e08: Pushed <br/>
> 98ff5a97582d: Pushed <br/>
> 3e207b409db3: Mounted from {dockerAccount}/alertmanager-configurer <br/>
> v1.1.0-master: digest: sha256:cb7abc2349a68935d3cdeb5fba184782a7bedd5688fe8f1a1465fa743cd2e361 size: 946<br/>
> Image pushed succesfully!!<br/>
> Error! Missing image! Please build the image<br/>

<b>Note:</b> there is no image for "grafana", this image will be create during the proccess of terraform scripts.

<pre>
 # cd /home/{user}/Magma/magma-1.1.0/symphony/app/fbcnms-projects/magmalte
 # docker-compose build magmalte
 ...
 Successfully built b96af63f4edc
 Successfully tagged magmalte_magmalte:latest
</pre>

<pre>
 # cp /home/{user}/Magma/magma-1.1.0/orc8r/cloud/docker/psw.txt .
 # COMPOSE_PROJECT_NAME=magmalte ../../../../orc8r/tools/docker/publish.sh -r docker.io/{dockerAccount} -i magmalte -v ${MAGMA_TAG} -u {dockerAccount} -p ./psw.txt
 ...
 v1.1.0-master: digest: sha256:8a6150b20dd7182fae17ffd8787151b2b8624ef06082d488fa140f4785aa0da0 size: 1789
 Image pushed succesfully!!
 
 # exit
</pre>
 
#### 5 - Create certs
###### PRE-REQUIRED
<p>You should have Certs Files: .crt .pem .key for your domain: yourdomain.com</p>
<p>If you already have these files, you can do the following:</p>
- Rename your public SSL certificate to controller.crt<br/>
- Rename your SSL certificate's private key to controller.key<br/>
- Rename your SSL certificate's root CA to rootCA.pem<br/>
- Put these 3 files under the directory you created above<br/>

<pre>
 $ mkdir -p ~/secrets/certs; cd ~/secrets/certs
        
 <b>In case you don't have the Certs File you can use self-sign certs, if this is your case use the command below</b>
 $ ~/Magma/magma-1.1.0/orc8r/cloud/deploy/scripts/self_sign_certs.sh yourdomain.com
        
 <b>This command will create the application certs:</b>
 $ ~/Magma/magma-1.1.0/orc8r/cloud/deploy/scripts/create_application_certs.sh yourdomain.com
</pre>
Ref: https://magma.github.io/magma/docs/orc8r/deploy_install

#### 6 - Create Github Access Token & Repository
 Github account:  {githubAccount}<br>
 Create the repository: magma
 
#### 7 - Create helm link
<pre>
 $ mkdir -p ~/Magma/helm; cd ~/Magma/helm
 $ helm package ~/Magma/magma-1.1.0/orc8r/cloud/helm/orc8r/
 Successfully packaged chart and saved it to: /home/{user}/Magma/helm/orc8r-1.4.21.tgz
 $ helm repo index .
 $ ll
</pre>

<pre>
 <b>OUTPUT:</b>
 drwxrwxr-x 2 {user} {group} 4.0K Sep  7 18:48 ./
 drwxrwxr-x 4 {user} {group} 4.0K Sep  7 18:48 ../
 -rw-r--r-- 1 {user} {group}  853 Sep  7 18:48 index.yaml
 -rw-rw-r-- 1 {user} {group}  32K Sep  7 18:48 orc8r-1.4.21.tgz
</pre>  

##### Initialized empty Git repository in /home/{user}/Magma/helm/.git/
<pre>
  $ git init
  $ git add .
  $ git config --global user.email your.email@your.company.com
  $ git config --global user.name {githubAccount}
  $ git commit -m "1.4.21"
  $ git remote add magma https://github.com/{githubAccount}/magma.git
  $ git push --set-upstream magma master
  <br>
  
  <b>OUTPUT:</b>
  Username for 'https://github.com': {githubAccount}
  Password for 'https://{githubAccount}@github.com': 
  Counting objects: 4, done.
  Delta compression using up to 2 threads.
  Compressing objects: 100% (4/4), done.
  Writing objects: 100% (4/4), 32.42 KiB | 10.81 MiB/s, done.
  Total 4 (delta 0), reused 0 (delta 0)
  To https://github.com/{githubAccount}/magma.git
   * [new branch]      master -> master
  Branch 'master' set up to track remote branch 'master' from 'magma'.
</pre>
 
##### Test Created helm link:
<pre>
 $ helm repo add magma 'https://raw.githubusercontent.com/{githubAccount}/magma/master/' --username {githubAccount} --password {your github token}              
 $ helm repo update
 $ helm repo list
</pre>
 Ref: https://blog.softwaremill.com/hosting-helm-private-repository-from-github-ff3fa940d0b7
 
 
#### 8 - Infrastructure and Application Installation (terraform)
<pre>
  $ cd ~/Magma/magma-1.1.0/orc8r/cloud/deploy/terraform/orc8r-helm-aws/examples/basic/
  $ cp main.tf main.tf.orig
</pre>        

##### Edit the following variables in the file main.tf:
<pre>
 deploy_nms: IMPORTANT - this should be false for now!
 nms_db_password     = "{Create_your_password}"
 orc8r_db_password   = "{Create_your_password}"
 orc8r_domain_name   = "subdomain.yourdomain.com.br"
 docker_registry     = "registry.hub.docker.com/{DockerAccount}"
 docker_user         = "{DockerAccount}"
 docker_pass         = "{password}"
 helm_repo           = "https://raw.githubusercontent.com/{githubAccount}/magma/master/"
 helm_user           = "{githubAccount}"
 helm_pass           = "{GitHubToken}"
 seed_certs_dir      = "~/secrets/certs"
                
 From: orc8r_chart_version = "1.4.7"
 To:   orc8r_chart_version = "1.4.21"

 From: orc8r_tag           = "1.0.1"
 To:   orc8r_tag           = "v1.1.0-master"
</pre>        
        
##### after edit and saving the main.tf file
<pre>
 $ terraform init
 $ terraform apply -target=module.orc8r
 $ mkdir -p ~/.kube/
 $ cp kubeconfig_orc8r ~/.kube/config
 $ terraform apply -target=module.orc8r-app.null_resource.orc8r_seed_secrets
 $ terraform apply
</pre>
        
##### kubectl
<pre>
 $ kubectl config set-context --current --namespace=orc8r
 $ export CNTLR_POD=$(kubectl get pod -l app.kubernetes.io/component=controller -o jsonpath='{.items[0].metadata.name}')
 $ kubectl exec -it ${CNTLR_POD} bash
	
 <b>The following commands are to be run inside the pod:</b>
 (pod)$ cd /var/opt/magma/bin
 (pod)$ envdir /var/opt/magma/envdir ./accessc add-admin -cert admin_operator admin_operator
 (pod)$ openssl pkcs12 -export -out admin_operator.pfx -inkey admin_operator.key.pem -in admin_operator.pem
           
 Enter Export Password:
 Verifying - Enter Export Password:
           
 (pod)$ exit
           
 $ cd ~/secrets/certs
 $ for certfile in admin_operator.pem admin_operator.key.pem admin_operator.pfx
 > do
 >    kubectl cp ${CNTLR_POD}:/var/opt/magma/bin/${certfile} ./${certfile}
 > done
      
 $ cd -
 $ terraform taint module.orc8r-app.null_resource.orc8r_seed_secrets
 $ terraform apply
</pre>

#### 9 - Final Application Terraform
<b> Edit main.tf </b>
<pre>
 From: deploy_nms          = false
 To:   deploy_nms          = true
</pre>

</br>

<pre>
 $ terraform apply
 $ kubectl exec -it $(kubectl get pod -l app.kubernetes.io/component=magmalte -o jsonpath='{.items[0].metadata.name}') -- yarn setAdminPassword master 

 {your.email@your.company.com} {your password}
 yarn run v1.22.4
 $ node -r '@fbcnms/babel-register' scripts/setPassword.js master {your.email@your.company.com} {your password}

 Creating a new user: email={your.email@your.company.com}, password={your password}
</pre>

#### 10 - DNS Resolution
 Use the information at AWS Route53 to create the CNAME at your main NS Record.
