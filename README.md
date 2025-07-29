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
