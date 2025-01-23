Below is an expanded README-style description that includes additional context, usage details, and some diagram examples (using Mermaid syntax) to illustrate how the components interact. You can copy-paste the Mermaid code blocks directly into GitHub or any other Markdown viewer that supports Mermaid.

AWS Generative AI & Boto3 Custom Layer

This repository demonstrates how to package a Python application and a custom AWS Lambda layer for use with AWS Boto3. It’s especially useful if you need a newer or custom version of Boto3 to access generative AI features (e.g., Amazon Bedrock or other ML/AI services) or if you simply want to decouple your Lambda code from the dependencies it requires.

Table of Contents
	1.	Overview & Motivation
	2.	Project Structure
	3.	High-Level Workflow
	4.	Architecture Diagrams
	5.	Usage Instructions
	6.	Local Development & Testing
	7.	Deploying the Custom Layer
	8.	License

Overview & Motivation
	•	Why a Custom Layer?
By default, AWS Lambda includes a version of Boto3. However, this version might be behind the latest features—particularly relevant if you are exploring Generative AI capabilities in AWS or advanced functionalities in new AWS services.
	•	Simplified Dependency Management
Keeping Boto3 (and its dependencies) in a separate layer helps you:
	1.	Reduce the size of your Lambda function deployment package.
	2.	Ensure your function can easily upgrade or downgrade Boto3 without changing your core code.
	3.	Cleanly separate your application logic (app.py) from underlying dependencies.

Project Structure

Below is the basic structure of the repository. The boto3_layer/ folder contains the unpacked Boto3 library and related dependencies.

```
siddharth-upadhyayula-aws-gen-ai/
├── README.md
├── app.py                <-- Main Lambda function code (entry point)
├── boto3_layer.zip       <-- Zipped layer that can be uploaded to AWS
├── requirements.txt      <-- Python dependencies list
└── boto3_layer/
    └── python/
        ├── boto3/
        ├── botocore/
        ├── dateutil/
        ├── jmespath/
        ├── s3transfer/
        ├── urllib3/
        └── ...
```
Key directories:
	•	app.py: Your actual Lambda function handler or main application logic.
	•	requirements.txt: Lists Python packages for use in the layer.
	•	boto3_layer/: Directory containing all dependencies (in the python/ subdirectory) that Lambda will treat as site-packages.
	•	boto3_layer.zip: The compressed artifact of the boto3_layer/ folder, used to create the custom layer in AWS.

High-Level Workflow
	1.	Create or Update requirements.txt: Ensure it has the exact versions of boto3, botocore, and other libraries you need.
	2.	Install Dependencies Locally: pip install -r requirements.txt -t boto3_layer/python.
	3.	Zip the Layer: cd boto3_layer && zip -r ../boto3_layer.zip python.
	4.	Upload the Layer: Use the AWS Console or AWS CLI to publish a new layer version from boto3_layer.zip.
	5.	Link the Layer to Lambda: In your Lambda function settings, add the newly created layer.
	6.	Deploy app.py: Zip your function code (if it’s a single file, you can zip app.py or deploy via your CI/CD pipeline) and upload it to the Lambda function.
	7.	Invoke: The Lambda function will automatically load Boto3, botocore, etc., from the layer instead of the default environment.

Architecture Diagrams

1. Flow Diagram

Below is a simple flow diagram showing how code & layers end up interacting in AWS:
```
flowchart LR
    A[Developer] -->|1. Build Layer<br>zip 'boto3_layer.zip'| B[Publish Layer to AWS]
    B -->|2. Attach Layer<br>to Lambda| C[AWS Lambda Function]
    C -->|3. Invoke<br>uses Boto3 from Layer| D[AWS Services <br>(S3, Bedrock, etc.)]
    D -->|Response| C
    C -->|Return Results| A
```
Explanation:
	1.	Developer creates the custom layer by installing dependencies and zipping them.
	2.	Publishes the layer to AWS, obtaining a Layer ARN.
	3.	Attaches the layer to the Lambda function’s configuration.
	4.	When invoked, the function loads Boto3 from the attached layer and interacts with AWS services.

2. Sequence Diagram

```
    participant Dev as Developer
    participant AWS as AWS
    participant LFunc as Lambda Function
    Dev->>AWS: Publish the Boto3 Layer (boto3_layer.zip)
    Dev->>AWS: Deploy Lambda function code (app.py)
    AWS->>LFunc: Attach Boto3 layer to Lambda
    Dev->>AWS: Invoke Lambda (test or production event)
    LFunc->>LFunc: Imports Boto3 from layer
    LFunc->>AWS: Calls AWS service (e.g. S3)
    AWS-->>LFunc: Service response
    LFunc-->>Dev: Return final output
```

Usage Instructions

1.	Clone this Repository
```
git clone https://github.com/your-username/siddharth-upadhyayula-aws-gen-ai.git
cd siddharth-upadhyayula-aws-gen-ai
```

2.	Install Dependencies (optional for local usage)

```
pip install -r requirements.txt
```

This is just for local testing. In actual practice, you’d install these into the boto3_layer/python/ folder for the layer.

3.	Build or Update the Layer

1.	Navigate to the layer folder:
```
cd boto3_layer
```

2.	Install the Python packages into python/:

```
pip install -r ../requirements.txt -t python
```

3.	Zip the folder:

```
zip -r ../boto3_layer.zip python
```

4.	(Optional) Return to the root directory:
```
cd ..
```

4.	Publish the Layer to AWS
Using AWS CLI, for example:
```
aws lambda publish-layer-version \
  --layer-name custom-boto3-layer \
  --zip-file fileb://boto3_layer.zip \
  --compatible-runtimes python3.8 python3.9 python3.10
```
This returns a LayerVersionArn, which you’ll use in your Lambda function config.

5.	Attach the Layer to Your Lambda
```
You can do this via:
	•	AWS Console: Go to your Lambda → Configuration → Layers → Add a layer → Select the custom layer you just created.
	•	AWS CLI:

aws lambda update-function-configuration \
  --function-name your-lambda-function \
  --layers "arn:aws:lambda:region:account-id:layer:custom-boto3-layer:version-number"
```

6.	Deploy app.py
1.	Zip your app.py (and any other function files):
```
zip function.zip app.py
```

2.	Upload to Lambda:

```
aws lambda update-function-code \
  --function-name your-lambda-function \
  --zip-file fileb://function.zip
```

	7.	Test/Invoke
Use the AWS Console, AWS CLI, or any event trigger to invoke your function. Example:
```
aws lambda invoke --function-name your-lambda-function output.json
cat output.json
```
Local Development & Testing
	•	Local Testing
You can test your app.py code locally by installing the same dependencies:
```
pip install -r requirements.txt
python app.py
```
(May need to mock AWS credentials or set them up properly via environment variables.)

	•	Integration Testing
For more realistic tests, consider using AWS SAM or LocalStack to emulate AWS services locally.

Deploying the Custom Layer

The typical steps to deploy or update the layer repeatedly:
	1.	Update requirements.txt to desired versions.
	2.	Reinstall into the boto3_layer/python folder.
	3.	Re-zip and re-publish the layer.
	4.	Update the Lambda function to reference the new version.

Repeat as needed whenever there’s a new version of Boto3, botocore, or other dependencies you’d like to pull in.

License

Different parts of this repository may come with their own licenses (e.g., Boto3, botocore, etc.). Review each .dist-info directory for the official license from the respective Python packages. Generally, these AWS-provided libraries fall under the Apache 2.0 license. If you add your own code (app.py, custom scripts, etc.), you can include a separate license statement for that portion (e.g., MIT, Apache 2.0, etc.).

Postman Tests:

<img width="892" alt="Screenshot 2025-01-20 at 3 10 35 PM" src="https://github.com/user-attachments/assets/396552fe-1117-4ea8-bd9c-bc347adfc83e" />

Generated Blog:

![Screenshot 2025-01-20 at 7 51 27 PM](https://github.com/user-attachments/assets/9f1d0d74-1d10-4e41-911f-c0b383723d40)
