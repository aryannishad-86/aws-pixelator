# aws-pixelator
# Stage 1 - Creating the S3 Buckets
We will create two buckets, both with the same name, but each suffixed with a functional title (see below). All settings, apart from region and bucket name, can be left as the default.
Click Create Bucket and create a bucket in the format of unique-name-source in the us-east-1 region
Click Create Bucket and create a another bucket in the format of unique-name-processed also in the us-east-1 region
These names will need to be unique, but as an example

Bucket 1 : dontusethisname-source
Bucket 2 : dontusethisname-processed

# Stage 2 - Creating the Lambda Role
Created a Lambda role: PixelatorRole with an inline policy
```{
	"Version": "2012-10-17",
	"Statement": 
	[
	  {
		"Effect":"Allow",
		"Action":[
		   "s3:*"
		],
		"Resource":[
			"arn:aws:s3:::pixelator-68-processed",
			"arn:aws:s3:::pixelator-68-processed/*",
			"arn:aws:s3:::pixelator-68-source/*",
			"arn:aws:s3:::pixelator-68-source"
		]
		
	  },
	  {
		  "Effect": "Allow",
		  "Action": "logs:CreateLogGroup",
		  "Resource": "arn:aws:logs:us-east-1:345918514093:*"
	  },
	  {
		  "Effect": "Allow",
		  "Action": [
			  "logs:CreateLogStream",
			  "logs:PutLogEvents"
		  ],
		  "Resource": [
			  "arn:aws:logs:us-east-1:345918514093:log-group:/aws/lambda/pixelator:*"
		  ]
	  }
	]
}
```
# Stage 3 - Creating a Lambda zip

From the CLI/Terminal Create a folder my_lambda_deployment
Move into that folder create a folder called lambda

Move into that folder Create a file called `lambda_function.py` and paste in the code for the lambda `pixelator` function (https://raw.githubusercontent.com/acantril/learn-cantrill-io-labs/master/00-aws-simple-demos/aws-lambda-s3-events/01_LABSETUP/lambda/lambda_function.py) then save

Download this file (https://files.pythonhosted.org/packages/f3/3b/d7bb231b3bc1414252e77463dc63554c1aeccffe0798524467aca7bad089/Pillow-9.0.1-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl) into that folder run unzip `Pillow-9.0.1-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl` and then `rm Pillow-9.0.1-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl`
These are the Pillow module files ... required for image manipulation in Python 3.9 (which is what the lambda function will be using)
From the same folder, run `zip -r ../my-deployment-package.zip .` which will create a lambda function zip, containing all these files in the parent directory.

# Stage 4 - Creating the Lambda Function
For Function name entered `pixelator`

For Runtime select `Python 3.9`

For Architecture selected `x86_64`

For `Permissions` expand `Change default execution` role pick `Use an existing role` and in the `Existing role` dropdown, pick `PixelatorRole`

Close down any `notifcation` dialogues/popups

Then upload the `.zip file` which we made in stage 3 from your local machine.
