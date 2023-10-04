# Jenkins-Zero-To-Hero

Are you looking forward to learn Jenkins right from Zero(installation) to Hero(Build end to end pipelines)? then you are at the right place. 

## Installation on EC2 Instance

YouTube Video ->
https://www.youtube.com/watch?v=zZfhAXfBvVA&list=RDCMUCnnQ3ybuyFdzvgv2Ky5jnAA&index=1


![Screenshot 2023-02-01 at 5 46 14 PM](https://user-images.githubusercontent.com/43399466/216040281-6c8b89c3-8c22-4620-ad1c-8edd78eb31ae.png)

Install Jenkins, configure Docker as agent, set up cicd, deploy applications to k8s and much more.

## AWS EC2 Instance

- Go to AWS Console
- Instances(running)
- Launch instances

<img width="994" alt="Screenshot 2023-02-01 at 12 37 45 PM" src="https://user-images.githubusercontent.com/43399466/215974891-196abfe9-ace0-407b-abd2-adcffe218e3f.png">

### Install Jenkins.

Pre-Requisites:
 - Java (JDK)

### Run the below commands to install Java and Jenkins
Run all these commands as a ubuntu user
Install Java

```
sudo apt update
sudo apt install openjdk-11-jre -y
```

Verify Java is Installed

```
java -version
```

Now, you can proceed with installing Jenkins, apply the complete command as is

```
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins -y
```
ps -ef | grep jenkins

you can check the jenkins installation using ps -ef | grep jenkins

**Note: ** By default, Jenkins will not be accessible to the external world due to the inbound traffic restriction by AWS. Open port 8080 in the inbound traffic rules as show below.

- EC2 > Instances > Click on <Instance-ID>
- In the bottom tabs -> Click on Security
- Security groups
- Add inbound traffic rules as shown in the image (you can just allow TCP 8080 as well, in my case, I allowed `All traffic`).

<img width="1187" alt="Screenshot 2023-02-01 at 12 42 01 PM" src="https://user-images.githubusercontent.com/43399466/215975712-2fc569cb-9d76-49b4-9345-d8b62187aa22.png">


### Login to Jenkins using the below URL:

http://<ec2-instance-public-ip-address>:8080    [You can get the ec2-instance-public-ip-address from your AWS EC2 console page]

Note: If you are not interested in allowing `All Traffic` to your EC2 instance
      1. Delete the inbound traffic rule for your instance
      2. Edit the inbound traffic rule to only allow custom TCP port `8080`
  
After you login to Jenkins, 
      - Run the command to copy the Jenkins Admin Password - `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`
      - Enter the Administrator password
      
<img width="1291" alt="Screenshot 2023-02-01 at 10 56 25 AM" src="https://user-images.githubusercontent.com/43399466/215959008-3ebca431-1f14-4d81-9f12-6bb232bfbee3.png">

### Click on Install suggested plugins

<img width="1291" alt="Screenshot 2023-02-01 at 10 58 40 AM" src="https://user-images.githubusercontent.com/43399466/215959294-047eadef-7e64-4795-bd3b-b1efb0375988.png">

Wait for the Jenkins to Install suggested plugins

<img width="1291" alt="Screenshot 2023-02-01 at 10 59 31 AM" src="https://user-images.githubusercontent.com/43399466/215959398-344b5721-28ec-47a5-8908-b698e435608d.png">

Create First Admin User or Skip the step [If you want to use this Jenkins instance for future use-cases as well, better to create admin user]

<img width="990" alt="Screenshot 2023-02-01 at 11 02 09 AM" src="https://user-images.githubusercontent.com/43399466/215959757-403246c8-e739-4103-9265-6bdab418013e.png">

Jenkins Installation is Successful. You can now starting using the Jenkins 

<img width="990" alt="Screenshot 2023-02-01 at 11 14 13 AM" src="https://user-images.githubusercontent.com/43399466/215961440-3f13f82b-61a2-4117-88bc-0da265a67fa7.png">

## Install the Docker Pipeline plugin in Jenkins:

   - Log in to Jenkins.
   - Go to Manage Jenkins > Manage Plugins.
   - In the Available tab, search for "Docker Pipeline".
   - Select the plugin and click the Install button.
   - Restart Jenkins after the plugin is installed.
   
<img width="1392" alt="Screenshot 2023-02-01 at 12 17 02 PM" src="https://user-images.githubusercontent.com/43399466/215973898-7c366525-15db-4876-bd71-49522ecb267d.png">

Wait for the Jenkins to be restarted.


## Docker Slave Configuration

Run the below command to Install Docker

```
sudo apt update
sudo apt install docker.io -y - Run this on same machine where jenkins is installed
you can also do it in another way installing docker on another ec2 machine but we need to configure the docker installed machine as a slave to the master jenkins

```
 
### Grant Jenkins user and Ubuntu user permission to docker deamon.

```
sudo su - 
usermod -aG docker jenkins  - granting the permissions to the jenkins users, to create/run containers and pull the images(jenkins basically comes with the user called jenkins user) - cat /etc/passwd
usermod -aG docker ubuntu - usermod is to grant the access to the jenkins, just making it to the part of docker group, docker installation creates a group called docker
systemctl restart docker
su - jenkins (we can run the docker commands on jenkins)- jenkins users is also about to create the containers and build the images as well
```

Once you are done with the above steps, it is better to restart Jenkins.

```
http://<ec2-instance-public-ip>:8080/restart
```

The docker agent configuration is now successful.

Differences
mvn clean install - if we wanna push enterprise archive, jar/war archive to the artifactory like nexus/jfrog.
mvn clean package - if we don't have a plan to push on to the artifactory, in my case I don't wanna  the the artifact anywhere, I just wanna use in the docker image and publish onto the image repository like dockerhub/ecr
mvn clean install or mvn clean package look for the pom.xml filepush
pom.xml in a nutshell - is responsible for getting the dependencies runtime and building the application
inside the target folder the artifact is stored

only use the jenkins-master for scheduling purpose and run actual worloads on jenkins-slaves(it's a traditional approach)

PROS:
use jenkins-master with docker as agents, will try to run the jenkins pipeline(different stages in a pipline) on docker conatiners, docker containers are ease to use, we can modify the dockerfile anytime we want with a desired configuration and can spin up the instance, at some cases we no need of create dockerfile, just we can pull the desired image from dockerhub, we can easily spin-up and tear down the docker conatiners, only if there is request a container is created
we used docker as agents in jenkins project setup in terms of cost and efficiency(spinning up and tearing down the docker containers when it's desired)

CONS:
in some cases, if your application is a database application this container approach might not work


docker typically run on a docker daemon process i.e., single-source-of-truth, and by default this daemon process isn't accessible by other users, so we need grant the access to docker daemon




