# Medivault-Day-4
Built a serverless event-driven medical image processing pipeline using AWS Lambda, S3, IAM, and CloudWatch.
This project simulates a real-world healthcare platform where hospitals and clinics upload medical images such as X-rays, MRI scans, and reports for automated serverless processing.

# Problem Statement

MediVault needed a scalable and automated cloud solution to:

automatically process uploaded medical images
eliminate manual backend processing
reduce operational overhead
improve scalability
enable real-time automation
avoid managing traditional servers
monitor processing workflows

The existing workflow required manual handling of uploaded files which slowed down processing and increased infrastructure complexity.

# Architecture
Doctors / Hospital Staff -> Upload Medical Image->S3 Upload Bucket->S3 Event Notification->AWS Lambda Trigger->Image Processing Workflow->Processed Images Bucket-CloudWatch Logs

<img width="1091" height="531" alt="day4 drawio" src="https://github.com/user-attachments/assets/740a1ecb-5cd5-4b7b-803c-62155c587a43" />

# Aws Services Used
IAM
Amazon S3
AWS Lambda
S3 Event Notifications
CloudWatch Logs

# Solution

Implemented a fully serverless event-driven architecture using Amazon S3 and AWS Lambda.

Whenever a medical image is uploaded into the S3 upload bucket:

S3 automatically triggers Lambda
Lambda processes the uploaded file
processed output gets stored into another S3 bucket
CloudWatch records execution logs and monitoring information

This removes the need for traditional backend servers and enables scalable real-time automation.

# Step 1: Created a new IAM user for MediVault cloud administration and enabled MFA
<img width="1916" height="898" alt="Screenshot 2026-05-23 232024" src="https://github.com/user-attachments/assets/caea41a2-1b71-42f3-9dba-5cdf4fbbb05c" />


# Step 2: Created IAM role for Lambda execution

<img width="1919" height="904" alt="Screenshot 2026-05-23 232645" src="https://github.com/user-attachments/assets/f21923b2-4e31-4ab9-8fe0-4f9fdccc570a" />

<img width="1919" height="911" alt="Screenshot 2026-05-23 232806" src="https://github.com/user-attachments/assets/b0e3a101-d201-429e-b66b-a6a845a8ef44" />


Attached permissions:
AWSLambdaBasicExecutionRole
AmazonS3FullAccess

# Step 3: Created two S3 buckets
Created:
medivault-medical-image-upload-prod

<img width="1917" height="903" alt="Screenshot 2026-05-24 002108" src="https://github.com/user-attachments/assets/0a27bc37-a2e2-4b51-965a-f670aa083ada" />

medivault-processed-images-prod

<img width="1918" height="973" alt="Screenshot 2026-05-23 232539" src="https://github.com/user-attachments/assets/4124541d-46b2-4621-a453-6010c5d92ffd" />

Purpose:
upload bucket stores original medical images
processed bucket stores processed outputs

# Step 4: Created Lambda function for serverless image processing

<img width="1919" height="910" alt="Screenshot 2026-05-23 233611" src="https://github.com/user-attachments/assets/94717f90-f076-4bef-be40-7c32a9271892" />


# Step 5: Added Lambda code to process uploaded images automatically
import json
import boto3
import urllib.parse

s3 = boto3.client('s3')

DEST_BUCKET = 'medivault-processed-images-prod'

def lambda_handler(event, context):
    for record in event['Records']:
        source_bucket = record['s3']['bucket']['name']
        source_key = urllib.parse.unquote_plus(
            record['s3']['object']['key']
        )
        copy_source = {
            'Bucket': source_bucket,
            'Key': source_key
        }
        s3.copy_object(
            Bucket=DEST_BUCKET,
            CopySource=copy_source,
            Key=source_key
        )
        print(f"Copied {source_key} to {DEST_BUCKET}")
    return {
        'statusCode': 200,
        'body': json.dumps('Image processed successfully')
    }
# Step 6: Configured S3 Event Notifications to automatically trigger Lambda

<img width="1916" height="919" alt="Screenshot 2026-05-23 234021" src="https://github.com/user-attachments/assets/da64c4dc-5ce5-4032-8a57-2fd12bf88e92" />


# Step 7: Uploaded test medical images into upload bucket

<img width="1915" height="908" alt="Screenshot 2026-05-23 234921" src="https://github.com/user-attachments/assets/0d794781-0253-484f-824b-883ed7a5bfef" />

After upload:

<img width="1919" height="916" alt="Screenshot 2026-05-23 235004" src="https://github.com/user-attachments/assets/2b6922a6-7a19-4946-9fca-e408e662ae43" />

Lambda executed automatically
image appeared in processed bucket

<img width="1919" height="908" alt="Screenshot 2026-05-23 235146" src="https://github.com/user-attachments/assets/42852d4b-c7e5-42da-8b53-0a750c8315dc" />

CloudWatch logs confirmed successful execution

# Step 8: Verified Lambda execution logs in CloudWatch

