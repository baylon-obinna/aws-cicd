END TO END CI/CD IMPLEMENTATION USING AWS CODEPIPELINE AND GITHUB

PREREQUISITE

- AWS account
  
- Github with source code
  
- Dockerhub account 

STEPS FOR CONTINUOUS INTEGRATION

- Login into aws in IAM service create a codebuild service role and attach these policies;

![image](https://github.com/baylon-obinna/aws-cicd/assets/111370498/472af47b-3a40-498a-96e2-29a201aa4266)

- Go to Aws system manager and create these parameters and their respective values;
  
![image](https://github.com/baylon-obinna/aws-cicd/assets/111370498/83504d8e-c497-4a16-b596-26877c641958)
this is to avoid hardcoding dockerhub senstive information in the source code.

- Edit the source code to have correct details of your parameters name as stored in the systems manager
  
- Go to Aws codebuild and create a project,assign the service role created in the first step, provide path to buildspec.yml (this file is what codebuild will use to orchestrate the build process
  
- Before starting the build give permissions for codebuild to build images through the environment settings

![image](https://github.com/baylon-obinna/aws-cicd/assets/111370498/178fa74c-cb03-4db1-b4cb-9d12ca6499d7)

- Start build

![image](https://github.com/baylon-obinna/aws-cicd/assets/111370498/5d30be3a-2fe9-4c36-ad08-306392d622a5)


Some errors while implementing continuous integration

![image](https://github.com/baylon-obinna/aws-cicd/assets/111370498/6cbcc8e6-ad14-4cac-be44-a26ec2d2ae46)
This error was as a result of the codebuild service role not having permission to Aws cloudwatch to create build logs for the build project. Fixed it by attaching AwsCloudwatch full access.

![image](https://github.com/baylon-obinna/aws-cicd/assets/111370498/aaff0b68-eb33-45aa-b6c2-231ed1cf0bd5)
This error was as a result of codebuild service not being able to access and call parameters and values from Aws systems manager. Fixed it by attaching AmazonSSMFullAccess and AWS systemsManagerForSAPFullAccess. {still figuring which of these did it}


STEPS FOR CONTINUOUS DEPLOYMENT/DELIVERY

- At the user interface under codebuild is codedeploy click on it and create an application
![image](https://github.com/baylon-obinna/aws-cicd/assets/111370498/f7c9dcad-7ae7-4dd5-bbc4-23d33251dc97)


- Go to Ec2 and create an instance, install docker and codedeploy agent from official documentation
  
- Create a codedeploy service role for Ec2 use case and attach these policies below for Ec2 and codedeploy to interact;
![image](https://github.com/baylon-obinna/aws-cicd/assets/111370498/096f4f53-ffc2-4a9b-b56d-3c380d61dde3)

- Associate the codedeploy service role to the Ec2 instance
  ![image](https://github.com/baylon-obinna/aws-cicd/assets/111370498/71af858b-3223-4248-878f-ebe8142a39fc)

- Create service role with CodeDeploy permissions that grants AWS CodeDeploy access to your target instances.
  ![image](https://github.com/baylon-obinna/aws-cicd/assets/111370498/60af0491-2620-4df8-a9f0-61f3f542c24f)

- Create a deployment group (target group), select the role above as service role, select Ec2 instance as environment configuration and attached the Ec2 instance created prior(loadbalancer was unchecked because for project just an instance was used).
  ![image](https://github.com/baylon-obinna/aws-cicd/assets/111370498/5f9416b6-40df-461d-bd77-26eb769cd6e0)

- Create a deployment specifying the deployment group, github repositry as source code with specific commit ID.
![image](https://github.com/baylon-obinna/aws-cicd/assets/111370498/1fa45ab8-ee80-4544-8abe-c2dbc56ff42b)

![image](https://github.com/baylon-obinna/aws-cicd/assets/111370498/4197f1ee-2733-4c13-a973-48e8bece81ad)


PIPELINE IMPLEMENTATION

- Click on pipeline below codedeploy
  
- Create pipeline, select v2 and new service role
  
- Select github as source provider and connect
   
- Select repository, branch and no filiter as trigger type
  
- Select codebuild as build provider, select project name created prior

- Select codedeploy as deploy provider

- Select application name and deployment created prior

- Create pipeline

  ![image](https://github.com/baylon-obinna/aws-cicd/assets/111370498/53197eda-ab35-4076-80c6-4a84ab76bea2)
Three stages of the pipeline all successfulERRORS WHILE IMPLEMENTING PIPELINE

![image](https://github.com/baylon-obinna/aws-cicd/assets/111370498/28dc9a1c-b496-4c17-82dd-6404ad2733ab)

![image](https://github.com/baylon-obinna/aws-cicd/assets/111370498/255f8945-9855-4798-bbcb-4d5797ada415)
This is an interesting error it kept me stranded for hours debugging, that error message doesn't really convey much so i had to dig into the deployment events,to see below.

![image](https://github.com/baylon-obinna/aws-cicd/assets/111370498/dbfe560a-8f06-4ef5-9147-5588489dba0e)
I had to add an if block to stopcontainer.sh to remove container/image if there's a container/image running due to the allocation of a specific port{5000} in the docker run command, also i recreated the ec2 instances because i felt the multiple failed build logs/data affected the health of the instance
