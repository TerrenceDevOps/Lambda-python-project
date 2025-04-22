
📂 S3 File Organizer with AWS Lambda
🧩 Problem Statement
In many real-world scenarios, clients upload large numbers of files into shared cloud storage solutions like Amazon S3. When left unmanaged, this flat file structure can quickly become disorganized, making it difficult to retrieve, analyze, or archive data efficiently.

Use Case:
A client continuously drops files into an S3 bucket we own. To maintain a clean and organized structure, we want to automatically sort these files into folders based on the date they were uploaded to S3.

The target folder structure is:

bash
Copy
Edit
s3://your-bucket-name/YYYY/MM/DD/filename
For example:

bash
Copy
Edit
s3://client-files/2024/12/28/invoice_1234.pdf
🚀 Solution Overview


We use an AWS Lambda function that gets triggered whenever a new file is uploaded to the S3 bucket. This function:

Reads the object metadata to determine the creation timestamp.

Parses the date into the format YYYY/MM/DD.

Copies the file to a new path in that folder.

Deletes the original file from the root of the bucket.

This ensures the S3 bucket remains automatically organized by upload date with zero manual intervention.

🧰 Tech Stack

Technology	Purpose
AWS S3	Object storage where files are dropped and organized
AWS Lambda	Serverless compute to react to new file uploads
IAM (AWS)	Secure roles and policies for permissions
Python 3.x	Lambda function logic
Boto3	AWS SDK for Python to interact with S3
Git & GitHub	Version control for our code
SSH	Secure GitHub access via key pairs
VS Code + Extensions	Development environment
🛠️ Setup Guide
1. Install Python & Boto3
bash
Copy
Edit
pip install boto3
Check Boto3: Boto3 Quickstart

2. AWS CLI Setup
Install and run:

bash
Copy
Edit
aws configure
Provide:

AWS Access Key

AWS Secret Key

Region

Output format (e.g., json)

3. Create the IAM User and Lambda Execution Role
Create an IAM user with programmatic access.

Assign AmazonS3FullAccess and AWSLambdaBasicExecutionRole to a custom role for the Lambda function.

4. Create the S3 Bucket
In the AWS Console, create a new bucket (e.g., client-files-bucket).

5. Write and Deploy the Lambda Function
python
Copy
Edit
# lambda_function.py

import boto3
import urllib.parse
from datetime import datetime

s3 = boto3.client('s3')

def lambda_handler(event, context):
    print("Event:", event)
    
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key = urllib.parse.unquote_plus(record['s3']['object']['key'])
        
        # Skip if file is already in a dated folder
        if len(key.split('/')) > 1:
            print(f"Skipping already organized file: {key}")
            continue

        response = s3.head_object(Bucket=bucket, Key=key)
        created_date = response['LastModified']
        folder_prefix = created_date.strftime('%Y/%m/%d')
        
        new_key = f"{folder_prefix}/{key}"
        
        # Copy to new location
        s3.copy_object(
            Bucket=bucket,
            CopySource={'Bucket': bucket, 'Key': key},
            Key=new_key
        )
        
        # Delete original object
        s3.delete_object(Bucket=bucket, Key=key)
        
        print(f"Moved {key} to {new_key}")
    
    return {'statusCode': 200, 'body': 'Success'}
6. Deploy Lambda
Zip your script and upload it to Lambda.

Attach the S3 trigger to the function.

Update the Handler to lambda_function.lambda_handler.

7. Test Upload
Upload a file directly to the root of your S3 bucket. The function should organize it automatically into the correct folder path.

📘 Learning Outcomes
✅ Hands-on experience with AWS Lambda for event-driven automation.

✅ Learned how to use S3 triggers to invoke Lambda functions.

✅ Mastered Boto3 for managing AWS services in Python.

✅ Understood IAM roles & security best practices.

✅ Built and deployed a serverless backend for real-world file management.

✅ Practiced code deployment & GitHub SSH key setup.

📝 Future Improvements
Add unit testing with pytest.

Log moved file names to CloudWatch or a database.

Skip re-processing files already organized.

Notify via SNS or Slack on errors.

🤝 Contributing
Feel free to fork this repo, suggest changes, or open issues for enhancements!

📬 Contact
Maintained by [TerrenceDevOps]
Email: terrencencube593@email.com
GitHub: https://github.com/TerrenceDevOps
