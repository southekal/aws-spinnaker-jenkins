### aws-spinnaker-jenkins setup

### spinnaker
- create IAM account
- install aws cli
- create ~/.aws/credentials
- install kubectl
- install halyard (linux) 
  - might need to create a file path and a user non-root
  - issue: `/usr/lib/sysusers.d/halyard.conf: No such file or directory`
  - `https://github.com/spinnaker/spinnaker/issues/4964`
  - manually create the path `/usr/lib/sysusers.d`
- start the aws EKS process to install spinnaker
  - `cloudformation` deploy in `https://www.spinnaker.io/setup/install/providers/kubernetes-v2/aws-eks/` runs into an error
  - `https://github.com/spinnaker/spinnaker.github.io/issues/1548`
  - update the kubernetes version to be > 1.10 (used 1.12) mentioned in `managing.yml`
- create managing and managed accounts through cloudformation on aws
- install `aws-iam-authenticator`  =.> `https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html`
- created `~/.kube/kubeconfig` 

env variables:

```
$VPC_ID 
vpc-0a525dfebc7027f72
$CONTROL_PLANE_SG 
sg-0395583a106ab83e2
$AUTH_ARN 
arn:aws:iam::544231248261:role/SpinnakerAuthRole
$SUBNETS 
subnet-080b4d25750d3d5f8,subnet-019e989c25f856a37
$MANAGING_ACCOUNT_ID 
544231248261
$EKS_CLUSTER_ENDPOINT 
https://AAF2069485B36DB980156CC660E293C4.sk1.us-west-2.eks.amazonaws.com
$EKS_CLUSTER_NAME 
spinnaker-cluster
$SPINNAKER_INSTANCE_PROFILE_ARN 
arn:aws:iam::544231248261:instance-profile/spinnaker-managing-infrastructure-setup-SpinnakerInstanceProfile-66CY2NOAHE45
```

```
hal config provider kubernetes account add ${MY_K8_ACCOUNT} --provider-version v2 --context $(kubectl config current-context)
looked like for eg:
hal config provider kubernetes account add spinnaker-system --provider-version v2 --context $(kubectl config current-context)
```

expose local installation of spinnaker with
`https://docs.armory.io/spinnaker/exposing_spinnaker/#exposing-spinnaker-on-eks-with-a-public-load-balancer`

load balance:
`UI_URL` => `http://aece0b7bb4eb011ea8a09068103f4adb-964213154.us-west-2.elb.amazonaws.com/#/search`
`API_URL` => `http://aece0b7bb4eb011ea8a09068103f4adb-964213154.us-west-2.elb.amazonaws.com/#/search`

- install jenkins as an aws service
`https://d1.awsstatic.com/Projects/P5505030/aws-project_Jenkins-build-server.pdf`
note: amazon linux t2.micro may have an older version of java installed for jenkins:
- `sudo yum install java-1.8.0`
- `sudo yum remove java-1.7.0-openjdk`
- jenkins url: `http://ec2-34-213-197-218.us-west-2.compute.amazonaws.com:8080/`

```
ssh -i ~/sandip-pixelbook.pem ec2-user@ec2-34-213-197-218.us-west-2.compute.amazonaws.com
```

- things with jenkins server:
- install openjdk 8 and maven version 3.6.1; naming should match the name provided in Jenkinsfile in the take home test
- install docker on the jenkins server and make sure its running
- ran into issues with existing test reports so tried
```
post {
                success {
                    #rm 'target/surefire-reports/**/*.xml'
                    junit 'target/surefire-reports/**/*.xml' 
                }
            }
```
in Jenkinfile

- to install maven on ec2 linux1 => set `export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk/jre` as 
`sudo yum install maven` was using java1.7 instead of java1.8
`https://phoenixnap.com/kb/how-to-install-apache-maven-on-centos-7`

- if you run into issues with jenkins being unable to connect to docker then do:
`sudo usermod -a -G docker jenkins` and restart
 
