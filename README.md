# Text-to-Speech Serverless App on AWS

## Project Overview
This is a serverless **Text-to-Speech (TTS) application** built on AWS using **Lambda** and **S3**. Upload a text file, and the Lambda function converts it into an audio file stored in an S3 bucket.  

## AWS Services Used
- **AWS Lambda** – Processes text and generates audio  
- **Amazon S3** – Stores input text and output audio  
- **Optional:** API Gateway – Trigger Lambda via HTTP  

## Architecture
1. User uploads a `.txt` file to S3 (`text2talk` bucket).  
2. S3 triggers a Lambda function.  
3. Lambda reads text → generates MP3 → stores in `text2talk-audio` bucket.  

## Setup Instructions
1. Create two S3 buckets: `text2talk` and `text2talk-audio`.  
2. Create a Lambda function in AWS Console or using AWS CLI.  
3. Add S3 trigger for `text2talk` bucket.  
4. Paste Lambda code (see `lambda_function.py`).  
5. Test by uploading a sample text file.  

## Lambda Function Example
import boto3
import os

s3 = boto3.client('s3')
polly = boto3.client('polly')

def lambda_handler(event, context):
    # Get environment variables
    target_bucket = os.environ['TARGET_BUCKET']
    voice_id = os.environ['VOICE_ID']
    output_format = os.environ['OUTPUT_FORMAT']
    
    # Process each record (S3 file upload)
    for record in event['Records']:
        source_bucket = record['s3']['bucket']['name']
        source_key = record['s3']['object']['key']
        
        # Get the uploaded text file content
        text_obj = s3.get_object(Bucket=source_bucket, Key=source_key)
        text = text_obj['Body'].read().decode('utf-8')
        
        # Use Polly to convert text to speech
        response = polly.synthesize_speech(
            Text=text,
            OutputFormat=output_format,
            VoiceId=voice_id
        )
        
        # Save the audio to the target bucket
        audio_key = source_key.rsplit('.', 1)[0] + '.' + output_format
        s3.put_object(
            Bucket=target_bucket,
            Key=audio_key,
            Body=response['AudioStream'].read()
        )
        
        print(f"✅ Converted {source_key} → {audio_key} and saved in {target_bucket}")
    
    return {
        'statusCode': 200,
        'body': 'Conversion completed successfully.'
    }

