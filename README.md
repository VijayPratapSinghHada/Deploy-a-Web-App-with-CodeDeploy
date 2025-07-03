<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Deploy a Web App with CodeDeploy

**Project Link:** [View Project](http://learn.nextwork.org/projects/aws-devops-codedeploy-updated)

**Author:** Vijay Pratap Singh Hada  
**Email:** vijaypratapsinghhada9@gmail.com

---

![Image](http://learn.nextwork.org/blissful_yellow_calm_donkey/uploads/aws-devops-codedeploy-updated_val-27)

---

## Introducing Today's Project!

In this project, I will demonstrate how to deploy a web application using AWS CodeDeploy. I'm doing this project to learn about continuous deployment and how to automate the process of getting new software versions onto servers.

### Key tools and concepts

Services I used were AWS CodeDeploy, AWS CodeBuild, and Amazon S3 for managing builds and deployments. Key concepts I learned include deployment automation, the importance of rollbacks, and how to handle deployment failures effectively to ensure service continuity.



### Project reflection

This project took me approximately 180 minutes to complete, including setup, deployment, and troubleshooting.

This project is part five of a series of DevOps projects where I'm building a CI/CD pipeline! I'll be working on the next project soon to continue enhancing my skills and apply what I've learned in a real-world scenario.

---

## Deployment Environment

To set up for CodeDeploy, I launched an EC2 instance and VPC because my web app needs a secure, isolated environment to run in production, separate from my development work. The EC2 instance is where my app will actually live for real users, and the VPC creates a private network to control how the app connects to the internet and other AWS services. 

Instead of launching these resources manually, I used AWS CloudFormation. This service lets me set up my EC2 instance and VPC using a template, so everything is created and connected the way I want with just one action. When I need to delete these resources, I can remove the whole stack in CloudFormation, and it automatically cleans up everything for me.

Other resources created in this template include a VPC, subnet, route tables, an internet gateway, and a security group, along with the EC2 instance. They're also in the template because a web app needs secure networking, internet access, and control over how it connects to other services. By including these resources, the template makes sure everything is set up safely and works together, making it simple to manage, repeat, or update the setup later.

![Image](http://learn.nextwork.org/blissful_yellow_calm_donkey/uploads/aws-devops-codedeploy-updated_val-5)

---

## Deployment Scripts

Scripts are small text files that contain commands to automate tasks you would usually type in the terminal one by one. To set up CodeDeploy, I also wrote scripts to automatically install all needed software, set the server settings, and make sure my web app runs smoothly. These scripts help CodeDeploy know exactly what steps to follow so my app can be set up and started without errors, saving time and making the process simple and repeatable.

The install_dependencies.sh script will set up all the software required to run my website. It installs Tomcat and Apache, which handle web traffic and host my application. The script also creates a configuration file that allows Apache to work with Tomcat, ensuring my website can be accessed by users on the internet. This way, everything is prepared automatically, making the deployment process more efficient and reliable.

The start_server.sh script will start both Tomcat and Apache, the servers needed to run my web application. It makes sure that Tomcat and Apache are up and running so users can access the website. Additionally, the script configures both services to automatically restart whenever the EC2 instance is rebooted, ensuring that my web app stays available at all times without needing manual intervention. This setup helps maintain a reliable and consistent experience for users visiting my site.

The stop_server.sh script will safely stop the web server services for Apache and Tomcat. It first checks if each service is currently running using `pgrep`. If it finds that Apache or Tomcat is active, it will then stop the service. This approach prevents errors by ensuring that the script only tries to stop services that are actually running, making it more reliable and robust. By checking the service status first, the script helps avoid unnecessary interruptions during the deployment process.

---

## appspec.yml

Then, I wrote an appspec.yml file to provide instructions for CodeDeploy on how to deploy my web application. The key sections in appspec.yml include version, which specifies the format version; os, indicating that we are deploying to a Linux system; files, which maps the source WAR file to its destination on the EC2 instance; and hooks, defining actions to run at specific stages during deployment, such as stopping the server before installing a new version, installing dependencies, and starting the server again. This file acts like a guide for CodeDeploy, ensuring everything happens in the right order during the deployment process, allowing for smooth and organized software updates.

I also updated buildspec.yml because I needed to include the appspec.yml file and the scripts folder in the artifacts section. This is important so that after CodeBuild compiles the application, it also packages these essential files for deployment. By adding appspec.yml and the scripts folder, I ensure that CodeDeploy has everything it needs to properly deploy and manage my application. Without these files, CodeDeploy wouldn't know how to handle the compiled code, making this update crucial for a successful deployment.

![Image](http://learn.nextwork.org/blissful_yellow_calm_donkey/uploads/aws-devops-codedeploy-updated_val-12)

---

## Setting Up CodeDeploy

A deployment group is a collection of EC2 instances grouped together to deploy an application simultaneously, allowing you to specify how and where the app will be deployed on those servers. It enables you to manage different environments, like testing and production, each with its own settings. A CodeDeploy application, on the other hand, serves as a main folder for your deployment project, organizing everything related to that application, such as deployment configurations and groups. In short, the deployment group focuses on the specific servers and deployment strategy, while the application acts as a container for the overall deployment process.

To set up a deployment group, you also need to create an IAM role to give CodeDeploy the necessary permissions to access and manage AWS resources on your behalf. This IAM role allows CodeDeploy to perform tasks such as deploying applications to EC2 instances, accessing application artifacts stored in S3 buckets, updating Auto Scaling groups, and logging activities in CloudWatch. By assigning this specific role, you ensure that CodeDeploy has the access it needs to operate effectively while maintaining security through limited permissions based on the principle of least privilege.

Tags are helpful for efficiently identifying and managing EC2 instances for deployment. I used the tag role: webserver to specify the purpose of the EC2 instance in my project. This allows CodeDeploy to easily find and deploy to the correct instance during the deployment process. Using tags also adds flexibility, as any new instances with the same tag will be automatically included in future deployments. Additionally, it serves as a self-documenting feature, making it clear what each instance does, and ensures seamless integration with the CloudFormation template we used earlier, which already tagged the EC2 instance appropriately.

![Image](http://learn.nextwork.org/blissful_yellow_calm_donkey/uploads/aws-devops-codedeploy-updated_val-18)

---

## Deployment configurations

Another key setting is the deployment configuration, which affects how quickly the application is rolled out to instances. I used CodeDeployDefault.AllAtOnce, so the application is deployed to all instances in the deployment group at the same time. This option allows for the fastest deployment, but it also carries more risk because if any issues arise, they could impact all instances simultaneously. For larger, production environments, more cautious options like OneAtATime or HalfAtATime are typically better choices, as they limit the potential impact of deployment problems. However, since we only have one instance for this project, the faster approach is more beneficial for our learning experience.

In order to connect my EC2 instance to CodeDeploy, a CodeDeploy Agent is also set up to facilitate communication between the instance and the CodeDeploy service. This software allows the instance to receive deployment instructions, such as shell scripts, from CodeDeploy and execute them accordingly. The agent plays a crucial role in ensuring that deployments are carried out smoothly and efficiently. Keeping the CodeDeploy Agent updated by scheduling regular updates every 14 days ensures that I have the latest features and fixes, improving the overall reliability of the deployment process.

![Image](http://learn.nextwork.org/blissful_yellow_calm_donkey/uploads/aws-devops-codedeploy-updated_val-20)

---

## Success!

A CodeDeploy deployment is a specific update to your application that includes a unique ID and detailed history. It defines which version of the application to deploy, where to deploy it (the deployment group), and how to carry out the deployment (the deployment settings). The difference between a deployment and a deployment group is that the deployment represents a single action that CodeDeploy takes to update the application, while the deployment group is a collection of EC2 instances that are targeted for that update. CodeDeploy manages the entire process of stopping services, copying files, running scripts, and restarting services, allowing you to track the progress and success of the deployment in real-time.

I had to configure a revision location, which means I specified where CodeDeploy should look for my application's build artifacts. This location helps CodeDeploy find the latest version of my web app to deploy to the EC2 instances. My revision location was the S3 bucket storing my ZIP file, allowing CodeDeploy to access the necessary files directly from there for the deployment process. This setup ensures that CodeDeploy retrieves the correct artifacts for a smooth and efficient deployment.

To check that the deployment was a success, I visited the Public IPv4 DNS of my EC2 instance in a web browser. I saw that my web application was now accessible, confirming that CodeDeploy completed the deployment process successfully. I also monitored the deployment status in CodeDeploy and ensured it showed as "Success," indicating that all the deployment steps, including the ones defined in my `appspec.yml`, were executed properly without any errors. 

![Image](http://learn.nextwork.org/blissful_yellow_calm_donkey/uploads/aws-devops-codedeploy-updated_val-27)

---

## Disaster Recovery

In a project extension, I decided to intentionally create an error in the stop_server.sh script to demonstrate how CodeDeploy handles failures. The intentional error I created was a misspelled command, changing systemctl to systemctll. This will cause the deployment to fail because the script will try to execute a command that doesn’t exist, resulting in a "command not found" error. This approach allows me to experience the deployment failure process and see how the rollback feature in CodeDeploy can quickly restore the application to its previous working state.

I also enabled rollbacks with this deployment, which means that if something goes wrong during the deployment, CodeDeploy will automatically revert to the last working version of the application. This feature is important because it helps minimize downtime and ensures that users can still access the application even if there is an error with the new update. By having rollbacks enabled, I can quickly recover from any deployment issues without having to spend a lot of time troubleshooting, allowing for a smoother and more reliable deployment process.

When my deployment failed, the automatic rollback feature would trigger to revert the application to the last known good version because it can quickly restore service without manual intervention. To actually recover from the error and ensure users do not experience downtime, I'd have to fix the issue in the code and deploy a new version of the application. In production environments, implementing automatic rollbacks is crucial, and this can be achieved by using tools like AWS CodePipeline. These tools can be configured to monitor deployments and automatically roll back to the last successful version whenever a failure is detected, ensuring a smoother and more reliable deployment process.

![Image](http://learn.nextwork.org/blissful_yellow_calm_donkey/uploads/aws-devops-codedeploy-updated_rollback-validation-upload)

---

---
