When you're working with any cloud provider, you want an easy way to connect to the cloud. You don't want to have to worry about creating custom scripts, API calls, and duct-tape workarounds just to deploy code or build infrastructure with Continuous Delivery.

Octopus Deploy has a clean and straight-forward way to connect to many cloud providers. In this blog post, you will learn how to connect Octopus Deploy to AWS

## Prerequisites

To follow along with this blog post, you want to ensure to have:

- An AWS account
- An Octopus Deploy server. On-prem or cloud are both viable options.

## Creating an IAM User

Before deploying from Octopus Deploy to AWS, there needs to be an authentication method. Because Octopus Deploy will be deploying infrastructure or applications to AWS, AWS needs to know *who* Octopus Deploy is. The typical authentication method for AWS is Identity and  Access Management ([IAM](https://aws.amazon.com/iam/#:~:text=AWS%20Identity%20and%20Access%20Management%20(IAM)%20enables%20you%20to%20manage,offered%20at%20no%20additional%20charge.)), which provides an access key and secret (sort of like a username and password).

### IAM User in the UI

1. To create an IAM user in the AWS UI, open up a web browser and go the [AWS console](https://aws.amazon.com/console/)

![](images/1.png)

2. Under **Find Services** in the search bar, type in **IAM**.

3. To access the place to create a new user, click on the **Users** option under **Access management**.

4. To create the new user that will have access to AWS from Octopus Deploy, click on the blue **Add user** button.

5. Under Set user details, create an appropriate name for the user name. For example - OctopusDeployAccount as shown in the screenshot below.

In the **Select AWS access type** section, you'll see two options:

- Programmatic Access
- AWS Management Console Access

You will want to select **Programmatic access** option because Octopus Deploy will be making API calls to AWS at the SDK level. When Octopus Deploy communicates with AWS, Octopus is not doing it from the AWS UI. Instead, it's making programmatic calls to AWS backend services, which requires access from the SDK level.

6. Once you choose **Programmatic access**, click the blue **Next: Permissions** button.

Permissions for the user are going to depend on what AWS services you want Octopus Deploy to have permissions to. For example, let's say you want Octopus Deploy to just deploy EC2 instances. In that case, you would give the IAM user access to something like `AmazonEC2FullAccess`. 

7. For the purposes of this blog post, since we want Octopus to have the ability to communicate with all services for AWS, we'll choose the `AdministratorAccess` policy under **Attach existing policies directly**. Once you choose the `AdministratorAccess` option, click the blue Next: Tags button as shown in the screenshot below.

8. Tags aren't necessary for the purposes of this blog post, so you can click the blue **Next: Review** button.

9. Finally, to create the new IAM user, click the blue **Create user** button.

You will be shown a screen that contains the Access key ID and the Secret access key. Ensure to save the Secret access key in a secure location because you will not be able to access it again. However, you can create a new secret access key if you lose this one. The Access key id and Secret access key will be used for the Octopus Deploy authentication.

![](images/10.png)

### IAM User on the CLI

As you saw in the previous section, creating an IAM user and adding them to the appropriate policy can be a bit cumbersome and a lot of clicking around. If you use the [AWS CLI](https://aws.amazon.com/cli/), there is a much easier way with just a few lines of code.

The first piece of code will be to create the new IAM user:

```
aws iam create-user --user-name OctopusDeployAWSAccount
```

The output should be similar to the screenshot below.

![](images/11.png)

Next, you'll need to create the Secret access key like we saw in the previous section. The Secret access key acts as a *password* of sorts.

To create the Secret access key, run the code below:

```
aws iam create-access-key --user-name OctopusDeployAccount
```

The output should look similar to the screenshot below.

![](images/12.png)

You are now ready to connect the AWS IAM account to Octopus Deploy.

## Connecting AWS to Octopus Deploy

In the previous section you learned about creating the ability to have Octopus Deploy interact with AWS at a programmatic level. Now that you understand the purpose of the access and secret keys, it's time to set up the AWS account in Octopus Deploy.

Open up a web browser and go to the Octopus Deploy server.

![](images/13.png)

Go to Infrastructure —> Accounts to set up the new AWS account.

![](images/14.png)

Under Accounts, click the green **ADD ACCOUNT** button and choose **AWS Account**.

![](images/15.png)

Under Details, you can add in some metadata about your account - the name and description as shown in the screenshot below.

![](images/16.png)

Next, under Credentials, you can add in the AWS access key and secret key, like in the screenshot below.

![](images/17.png)

Finally, you can set restrictions under the Restrictions section. For example, I've chosen to allow this account for my Dev environment only.

![](images/18.png)

Once complete, click the green **SAVE** button on the top of the page.

![](images/19.png)

Congrats! You have successfully set up an AWS account in Octopus Deploy.

## Conclusion

The need to interact with different cloud-based platforms is not a need that will go away. Regardless of what Continuous Delivery and Deployment tool you're using, there will always be a reason for authentication from the CD tool to a cloud, or even on-prem, platform. 

In this blog post, you learned about not only how to connect Octopus Deploy to AWS, but the different authentication methods available for AWS.
