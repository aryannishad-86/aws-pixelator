# aws-pixelator
# Stage 1 - Creating the S3 Buckets
We will create two buckets, both with the same name, but each suffixed with a functional title (see below). All settings, apart from region and bucket name, can be left as the default.

Click Create Bucket and create a bucket in the format of unique-name-source in the us-east-1 region

Click Create Bucket and create another bucket in the format of unique-name-processed, also in the us-east-1 region

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

From the CLI/Terminal, create a folder my_lambda_deployment
Move into that folder, create a folder called lambda

Move into that folder. Create a file called `lambda_function.py` and paste in the code for the lambda `pixelator` function 
```import os
import json
import uuid
import boto3

from PIL import Image

# bucketname for pixelated images
processed_bucket=os.environ['processed_bucket']

s3_client = boto3.client('s3')

def lambda_handler(event, context):
	print(event)
	
	# get bucket and object key from event object
	source_bucket = event['Records'][0]['s3']['bucket']['name']
	key = event['Records'][0]['s3']['object']['key']
	
	# Generate a temp name, and set location for our original image
	object_key = str(uuid.uuid4()) + '-' + key
	img_download_path = '/tmp/{}'.format(object_key)
	
	# Download the source image from S3 to temp location within execution environment
	with open(img_download_path,'wb') as img_file:
		s3_client.download_fileobj(source_bucket, key, img_file)
		
	# Biggify the pixels and store temp pixelated versions
	pixelate((8,8), img_download_path, '/tmp/pixelated-8x8-{}'.format(object_key) )
	pixelate((16,16), img_download_path, '/tmp/pixelated-16x16-{}'.format(object_key) )
	pixelate((32,32), img_download_path, '/tmp/pixelated-32x32-{}'.format(object_key) )
	pixelate((48,48), img_download_path, '/tmp/pixelated-48x48-{}'.format(object_key) )
	pixelate((64,64), img_download_path, '/tmp/pixelated-64x64-{}'.format(object_key) )
	
	# uploading the pixelated version to destination bucket
	upload_key = 'pixelated-{}'.format(object_key)
	s3_client.upload_file('/tmp/pixelated-8x8-{}'.format(object_key), processed_bucket,'pixelated-8x8-{}'.format(key))
	s3_client.upload_file('/tmp/pixelated-16x16-{}'.format(object_key), processed_bucket,'pixelated-16x16-{}'.format(key))
	s3_client.upload_file('/tmp/pixelated-32x32-{}'.format(object_key), processed_bucket,'pixelated-32x32-{}'.format(key))
	s3_client.upload_file('/tmp/pixelated-48x48-{}'.format(object_key), processed_bucket,'pixelated-48x48-{}'.format(key))
	s3_client.upload_file('/tmp/pixelated-64x64-{}'.format(object_key), processed_bucket,'pixelated-64x64-{}'.format(key))
	
def pixelate(pixelsize, image_path, pixelated_img_path):
	img = Image.open(image_path)
	temp_img = img.resize(pixelsize, Image.BILINEAR)
	new_img = temp_img.resize(img.size, Image.NEAREST)
	new_img.save(pixelated_img_path)
```
then save

Download this file (https://files.pythonhosted.org/packages/f3/3b/d7bb231b3bc1414252e77463dc63554c1aeccffe0798524467aca7bad089/Pillow-9.0.1-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl) into that folder run unzip `Pillow-9.0.1-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl` and then `rm Pillow-9.0.1-cp39-cp39-manylinux_2_17_x86_64.manylinux2014_x86_64.whl`
These are the Pillow module files ... required for image manipulation in Python 3.9 (which is what the lambda function will be using)
From the same folder, run `zip -r ../my-deployment-package.zip .` which will create a lambda function zip, containing all these files in the parent directory.

# Stage 4 - Creating the Lambda Function
For Function name, enter `pixelator`

For Runtime select `Python 3.9`

For Architecture selected `x86_64`

For `Permissions` expand `Change default execution` role, pick `Use an existing role` and in the `Existing role` dropdown, pick `PixelatorRole`

Close down any `notification` dialogues/popups

Then upload the `.zip file` which we made in stage 3 from your local machine.

# Stage 5 - Configuring the Lambda Function & Trigger
Configuring the `Environment Variable`

We need to add an environment variable telling the pixelator function which processed bucket to use, it will know the source bucket because it's told about that in the event data.

Under `Key` put `processed_bucket` and for `Value` put the bucket name of your processed bucket.

`Edit` the `General Configuration` and change the timeout to `1` minutes and `0` seconds.

Adding Trigger 

Click `Add trigger`

In the dropdown, pick `S3`

Under `Bucket`, pick your source bucket. Only pick your SOURCE bucket here.

# Stage 6 - Test and Monitor
Open a tab to the `CloudWatch logs` console.

Make sure you have two tabs open to the `S3` console. In one tab, open your `-source` bucket & in the other open the `-processed` bucket.

In the `-source` bucket tab, make sure to select the `Objects` tab and click `Upload`. Add some files and click `Upload`

https://github.com/user-attachments/assets/2eab37fc-dcba-4365-84b7-d571344fb973

