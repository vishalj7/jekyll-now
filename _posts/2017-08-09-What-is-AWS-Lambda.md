---
layout: post
title: What is AWS Lambda?
tags: AWS Lambda
---


### What is AWS Lambda?
It is a function in the AWS suite of tools which allows you to run code without setting up and running servers. You can set up AWS Lambda to excute your code when you want based on a event or schedule it. AWS Lambda supports a number of languages such as Java, Python, C# and Node.js.



### AWS Lambda Running in Response to Events

As mentioned AWS Lambda can be triggered to run your code based on an event, this event can be changes to the state of your AWS S3 bucket or changes to the data in AWS Dynamo DB. 

When you create a new AWS Lambda function you can search for existing code and use that or you can create your own code and use that instead. If your code doesn't need any external packages then it is best to use the 'Edit Code Inline'. Otherwise you need to zip your code and any necessary packages/libraries together and the upload the zip file to your AWS Lambda. 


### How can I use it?

I will now go through the process to set up and create the zip required to upload your code to the AWS Lambda function. 

First, you need to create your code on your local system and once that is done, start to zip your code and packages/libraries together. 

For Python there is a tool called 'virtualenv' which creates a envrionment for you where you can install the packages you need to zip your code together with for AWS Lambda.

Depending on which Python version you hae either use pip3 if using Python 3 or pip if using Python 2. 

#### Note you would need to install pip first before using the commands below.


### Below is the process to upload a zip file to AWS Lambda

The command below is to install the virual environment and is a one off process.

```bash
# Install the virtualenv tool
pip3 install virtualenv
```

The next commands give a guide on the steps to create the zip file

```bash

# Select an area for where we can create a environment
virtualenv path/to/my/helloworld-env      

# Specifiy the path for where you created the environment - this will start the environment and you can now start installing the packages/library dependencies you need
source path/to/my/helloworld-env/bin/activate

# When you are installing the packages you require make sure that you are in the environment you created above, you will know this as it will have the environment name before the start of the command line - eg . (helloworld-env) vishal@vishal-VirtualBox:~$ 

# Once you have completed installing the packages you then zip all the packages and python scripts together which can then be uploaded to AWS Lambda.
```

I have set up my AWS Lambda function to run when an image is put/uploaded into a S3 bucket. The function waits for this event and then its connects to my EC2 which  copies the images to another bucket and then calls the rekognition API to analyse the image. 

Below are the steps in full and the packages I used to zip the files together.

```bash
# As above I created my environment and started to install my packages
pip3 install pycrypto

pip3 install paramiko

# In case installing the above package causes error run the below command:
sudo apt-get install build-essential libssl-dev libffi-dev python-dev

# Next we zip the Python scripts to a zip file which we will later upload to AWS Lambda
zip path/to/zip/zip_filename.zip python_scripts.py


# Now we zip the package files from the environment we created. First cd to the environment area
cd path/to/my/helloworld-env/lib/python3.5/site-packages

# Then zip all the packages and its dependencies to the zip file (Note the . instead of * as there might be some hidden files)
zip –r9 path/to/zip/zip_filename.zip .

# There may be other packages and its dependencies in another folder (lib64) so the below commands are to cover them 
cd path/to/my/helloworld-env/lib64/python3.5/site-packages

# Same as before 
zip –r9 path/to/zip/zip_filename.zip .
```

Now you are ready to upload the zip file to AWS Lambda before you can test the script you need to tell AWS Lambda where you main function is that it needs to call. 

You click on configuration tab and set the Handler to be the filename.method_name. Once that is done, you can test your code.

One issued I faced was that the function was failing and the error was saying that there is a package/dependency was missing. It was not clear why the package/dependancy was missing as it was part of the packages I installed and it was working when I tested the code on my local machine. I then had to copy the missing package/dependancy from the /lib/x86_64-linux-gnu/ this then solved the problem.    

And that's it now my AWS Lambda function will run when an image is put/uploaded into my S3 bucket.
