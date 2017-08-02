# CloudFormation Alexa Python

**This readme is not yet complete, and the installation does not currently work!**

### Prerequisites

Before getting started with this project, you will require:

* An Amazon Echo, Echo Dot or Echo Tap (or be willing to use the Amazon simulator.)
* An installation of Python 2.7 on your device, found [here](https://www.python.org/downloads/).
* An Amazon Web Services account.
* An Amazon Developer account.
* Some pre-obtained knowledge on the functionality of AWS [Lambda](https://aws.amazon.com/lambda/) and [Alexa Skills Kit](https://developer.amazon.com/edw/home.html#/skills).

It is also advised to install the necessary module [virtualenv](https://virtualenv.pypa.io/en/stable/), as this will help during the deployment process. This can be achieved by:

```
setx PATH "%PATH%;C:\Python27\Scripts" (Windows only - Optional)

pip install virtualenv
```

Note: The first line enables 'pip' to be used to install Python modules, and you can skip this line if you have used 'pip' on your machine previously. Additionally, if an error occurs, you may need to close and reopen your CMD terminal window after entering the first line before installing virtualenv.

## Getting Started

First of all, you will need to create an S3 bucket on Amazon Web Services ([AWS](https://aws.amazon.com/)), with permissions available only to yourself, and not public. This bucket will contain user-data, such as phone numbers for Two-Factor Authentication, and so this is important. You will then need to create a directory on your local device, such as "Alexa_Skill", and download the [lambda_function.py file](https://github.com/capgemini-psdu/cloud-former-alexa/blob/master/cloud-former-lambda/Python/tempf/lambda_function.py) to that location. It is important that the filename is not altered throughout this process.

## Deployment

There are two methods of deploying the function to AWS Lambda. The first is manually, using [this guide](http://docs.aws.amazon.com/lambda/latest/dg/lambda-python-how-to-create-deployment-package.html). Alternatively, the automated and simplified deployment service [Zappa](https://github.com/Miserlou/Zappa) can be used. If you choose to use Zappa, a detailed guide can be found on the corresponding [Github](https://github.com/Miserlou/Zappa). Alternatively, this readme will demonstrate the manual method, although this method can be more difficult.

First, navigate to the directory where the lambda_function.py file is stored. Open the file, and then modify the constants at the top of the file with the name and region of your S3 bucket. For example:

```
userbucketname='s3-bucket-name-example'
userbucketregion='eu-west-1'
```
Save and exit the file. Then, create a Python Virtual Environment in the directory you made (eg Alexa_Skill) by
```
virtualenv virtual-env
```
where 'virtual-env' is the name of the environment. The name of this can be user-specified, and can vary however needed.

Then, activate the virtual environment by (for Windows):
```
virtual-env\Scripts\activate.bat
```
or (for macOS):
```
source  virtual-env/bin/activate
```

If this is successful, the environment name (virtual-env) will appear on the left in the terminal window, to inform you that you are based within the virtual environment. Next, install the following modules:

```
pip install boto3
pip install botocore
pip install flask
pip install flask_ask
```

**There is currently a bug within the Python module flask_ask, specifically regarding _openssl. This is [being worked on](https://github.com/johnwheeler/flask-ask/issues/167).**

(Note, installing boto3 is optional and should already function on AWS Lambda. However, it has been stated here for completeness.)

Now, to create a deployment package you do the following:

* First, create .zip file with your Python code you want to upload to AWS Lambda.
* Add the libraries from preceding activated virtual environment to the .zip file. That is, you add the content of the following directory to the .zip file (note that you add the content of the directory and not the directory itself).

Location (for Windows):
```
virtual-env\Lib\site-packages
```
or (for macOS):
```
virtual-env/lib/python2.7/site-packages
```

This .zip file will then need to be uploaded to AWS Lambda.

### AWS Lambda

(Note, if you are using Zappa to deploy this function, you can skip any steps not in **bold**.)

To create the Lambda function:

* Navigate to the [Lambda Website](https://aws.amazon.com/lambda/), sign in, and then choose to create a Lambda function.
* Opt for a blank function template.
* For a trigger, choose 'Alexa Skills Kit'.
* For Name and description, specify any input you prefer.
* Runtime: Python 2.7.
* Code entry type: Choose to upload a .zip file.
* Role: You will need to create this role separately.

In a new window, navigate to [AWS IAM Roles](https://console.aws.amazon.com/iam/home#/roles):

* Create a custom role.
* Choose AWS Service Role > AWS Lambda.
* Enable EC2FullAccess, VPCFullAccess, SNSFullAccess, CloudWatchFullAccess.
* Choose next, and give the role a name, and create the role.

**When the role is created, within the role** (this will still need to be applied to the role automatically created by Zappa, if you use that method):

* Click on 'Create Role Policy' or 'Add inline policy'.
* Navigate to policy generator.
* Choose AWS Service: AWS CloudFormation.
* Actions: All Actions.
* ARN Name: *
* Click on 'Add Statement'.
* Click on 'Next Step', and then 'Apply Policy'.

Now navigate back to the Lambda function.

* For role, select the role you just created.
* Advanced settings: Change the timeout to be at least 10 seconds or greater, to allow the necessary Lambda function to complete. (Note, the maximum timeout is set to 5 minutes by Amazon directly.)
* Review and create the Lambda function.

The Lambda function is now created and ready.

### Amazon Alexa Skill Setup

Within your Amazon Developer Portal, navigate to the [Alexa Skills Kit](https://developer.amazon.com/edw/home.html#/skills). Then:

* Add a new skill.
* Change the language to the relevant location. (Note: This skill was tested with English UK.) (This language CANNOT be changed at a later date.)
* Choose a name for the skill, and an invocation name. This is what users will say to initiate your skill.
* For the 'Intent Schema', copy and paste the text in the file /Python/CloudFormation_Templates/intentschema.json on [Github](https://github.com/capgemini-psdu/cloud-former-alexa/blob/master/cloud-former-lambda/Python/CloudFormation_Templates/intentschema.json).
* For the 'Sample Utterances', copy and paste the text in the file /Python/CloudFormation_Templates/sampleutterances.txt on [Github](https://github.com/capgemini-psdu/cloud-former-alexa/blob/master/cloud-former-lambda/Python/CloudFormation_Templates/sampleutterances.txt).
* In 'Custom Slot Types', create a slot called 'user', and enter the names of the users who you wish to have access to the Two-Factor Authentication codes. For example, you could use first names, such as 'John', or 'Bethany'.
* In configuration, choose AWS Lambda ARN, and then paste in the ARN and geographical region of your Lambda function.

### Testing

In the testing panel, type:

"What is the date."

and the skill should respond with:

"date/month"

if the skill is functioning. If you receive an error, investigate the CloudWatch logs and diagnose accordingly.

**There is currently a bug in the Alexa simulator, which will cause this skill to fail. To counter this, write a request in written English, copy the corresponding JSON request and then re-send that, as a temporary workaround.**

## Additional Requirements

You will require at least one CloudFormation template in your S3 bucket. An example of this can be found in /Python/CloudFormation_Templates/basic_ec2_instance.json on [Github](https://github.com/capgemini-psdu/cloud-former-alexa/blob/master/cloud-former-lambda/Python/CloudFormation_Templates/basic_ec2_instance.json). This will launch a Linux EC2 instance within the Free Tier of AWS.

Furthermore, you will need a file entitled 'contacts.csv', in the S3 bucket, in the format:

```
john,+44XXXXXXXXXX
bethany,+1XXXXXXXXX
```

**It is vital that the names match those on the 'Custom Slot Types' specified when setting up the Alexa Skill.**

## Features and Functionality

The following assumes the invocation name is "Cloud".

### Initiating/Resetting the Skill

*	“Alexa, ask Cloud to…” for a specific question, or: “Alexa, launch Cloud” if you do not have a specific question.
(You can invoke this at any time if the skill pauses, and it will remember where you left off.)
*	To reset the conversation, say “Alexa, ask Cloud to reset skill.”

### Creating a Stack

(Alexa responses are in *italics*.)

(**The skill will NOT let you launch a stack without asking for available templates beforehand - to prevent you launching an incorrect and potentially expensive template.**)

*	“(Alexa, ask Cloud to) List available templates.” (**Must do this the first time you use the skill.**)
*	*“1. Basic instance. 2. Secondary instance.”*
*	“Launch stack” / “Start stack” / “Launch stack (number)”. (Specifying a number is optional.)
*	*“Which stack would you like to launch?”*
*	“One” / “Number one” / “Stack one”.
*	*“Please specify your name/username.”*
*	“Jon” / “Jordan” / “(username)”.
*	*“You have been sent a 2FA code to your phone. Say that code now.”*
*	“One two three four” / “(code)”. (There is a 60-second period to do this step.)
*	*“Stack (number) has been launched.”*

### Deleting a Stack

(Alexa responses are in *italics*.)

*	“(Alexa, ask Cloud to) List all stacks.”
*	*“Name: Cloud-Former-1. Status: CREATE COMPLETE. Name: Cloud-Former-2. Status: CREATE COMPLETE. ……”*
*	“Terminate stack” / “Delete stack” / “Delete stack (number)”. (Specifying a number is optional.)
*	*“Which stack would you like to delete?”*
*	“One” / “Number one” / “Stack one”.
*	*“Please specify your name/username.”*
*	“Jon” / “Jordan” / “(username)”.
*	*“You have been sent a 2FA code to your phone. Say that code now.”*
*	“One two three four” / “(code)”. (There is a 60-second period to do this step.)
*	*“Stack (number) has been deleted.”*

### Describing a Specific Stack

(Alexa responses are in *italics*.)

*	“Alexa, ask Cloud to list Stack Status (number).”
*	*“(Stack status and resources).” Or “That stack either does not exist, or has been deleted.”*

### Estimating Template Costs

(Alexa responses are in *italics*.)

*	“Alexa, ask Cloud how much stack (number) will cost.”
*	*“Please specify your name/username.”*
*	“Jon” / “Jordan” / “(username)”.
*	*“The cost URL has been sent to your mobile device.”*

This URL directs you to AWS, which will contain the estimated monthly cost of the instance to be launched.

## Debugging

If you find the skill is not performing as expected, navigate to the Alexa Skill simulator within your [Alexa Skills Kit](https://developer.amazon.com/edw/home.html#/skills). This way, you will be able to see the response returned by the Lambda function. If the response returned is

```
There was an error calling the remote endpoint, which returned (error here).
```
or
```
The response is invalid.
```

then viewing the CloudWatch log for your Lambda function should help diagnose why the skill failed.

If any help is required, please contact the developer for this skill.

## Built With

* [Flask](http://flask.pocoo.org/) - A microframework for Python.
* [Flask-Ask](https://github.com/johnwheeler/flask-ask) - Flask extension used to simplify the Python code when building the Alexa skill.

## Authors

* **Jordan Lindsey** - *Initial work* - [Github](https://github.com/jlindsey1)

See also the [main repository](https://github.com/capgemini-psdu/cloud-former-alexa) for all those who participated in this project.

## License

Flask is Copyright (c) 2015 by Armin Ronacher and contributors. Some rights reserved.

Flask-Ask is licensed under the Apache License 2.0.

This project is Copyright (c) 2017 by Capgemini UK.

## Acknowledgments

* The main nodeJS version of this skill can be found [here](https://github.com/capgemini-psdu/cloud-former-alexa/tree/master/cloud-former-lambda/nodeJS).