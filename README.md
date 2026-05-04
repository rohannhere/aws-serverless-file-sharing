# 📂 Serverless File Sharing Platform

The Serverless File Sharing Platform allows users to securely upload and download files via a simple HTTP API. It leverages AWS Lambda for serverless compute, API Gateway for RESTful API management, and Amazon S3 for durable and scalable object storage. Users can interact with the platform using any HTTP client (like Postman), making it versatile for various use cases that involve file sharing and storage.

---

### Use Cases
- Users can upload files to the platform using a POST request, specifying the file name and content. This can be useful for sharing documents, images, or any other digital assets securely.
- Teams can share project resources, documents, and data securely, facilitating collaboration across different locations.

---

## 🏗️ Architecture & Technologies Used
*   **Amazon API Gateway:** Acts as the entry point, providing RESTful API endpoints for client requests.
*   **AWS Lambda (Python 3.9):** Serverless compute handles the business logic (parsing requests, encoding/decoding, and communicating with S3).
*   **Amazon S3:** Provides durable, scalable object storage for the uploaded files.
*   **IAM:** Secures the architecture using least-privilege execution roles for the Lambda functions.
*   **Client Testing:** Postman & `curl` (Command Line Interface).

---

## 🚀 Key Features
*   **POST Endpoint:** Users can upload files by passing query string parameters and raw data payloads.
*   **GET Endpoint:** Users can retrieve files, which are returned as Base64 encoded strings to preserve data integrity during HTTP transport.
*   **Serverless:** Zero servers to patch, manage, or scale.

---

## 🛠️ Setup & Deployment

*(Note: To replicate this project, you must have an active AWS Account.)*

### Step 1: Create the S3 Bucket (Storage)
This bucket will hold all the files uploaded by users.

- Navigate to the S3 Console in AWS.
- Click Create bucket.
- Bucket name: Enter a globally unique name (e.g., my-file-share-buck-623). Note this name, you will need it later.
- AWS Region: Choose your preferred region (e.g., ap-south-1).
- Keep all other settings as default (Block all public access should remain ON for security, as our Lambda function will handle the access).
- Click Create bucket.

---

### Step 2: Configure IAM Execution Roles

Your Lambda functions need permission to interact with S3 and CloudWatch (for logging).

- Navigate to the IAM Console.
- Click Roles on the left menu, then Create role.
- Trusted entity type: Select AWS service.
- Use case: Select Lambda, then click Next.
- Add permissions: Search for and attach the AWSLambdaBasicExecutionRole (this allows logging to CloudWatch).
- Click Next, name the role FileShareLambdaRole, and click Create role.
- Find your newly created role in the list and click on it.
- Under the Permissions tab, click Add permissions -> Create inline policy.
- Select the JSON tab and paste the provided policy (replace YOUR_BUCKET_NAME with the exact name of your bucket):

---

### Step 3: Create the Lambda Functions (Compute)

We will create two separate functions: one for uploading, one for downloading.

#### 3A. The Upload Function

- Navigate to the Lambda Console and click Create function.
- Select Author from scratch.
- Function name: upload_function.
- Runtime: Select Python 3.10 
- Execution role: Select Use an existing role and choose the FileShareLambdaRole you created in Step 2.
- Click Create function.
- In the Code Source section, paste your Python upload script (ensure the bucket name variable in the code matches your actual bucket).
- Click Deploy.

#### 3B. The Download Function

- Go back to the Lambda dashboard and click Create function again.
- Function name: download_function.
- Runtime: Python 3.10
- Execution role: Select the exact same FileShareLambdaRole.
- Click Create function.
- Paste your Python download script (handling the S3 get object and Base64 encoding).
- Click Deploy.

---

### Step 4: Configure API Gateway (The Entry Point)

- Name: file-sharing-api
- Create two resources: /files with POST and GET methods.
- For each method, configure Lambda integration with UploadFunction and DownloadFunction respectively.

---

### Step 5: Configure GET Method:

- Method Request --> Edit --> Request validator --> Validate Query String Parameters and Headers
- Method Request --> Edit --> Request Body --> text/plain
- Integration Request --> Edit --> Mapping Templates --> Content Type: application/json --> Content Body:
```bash
{
  "queryStringParameters": {
      "fileName": "$input.params('fileName')"
  }
}
```
---

### Step 6: Configure POST Method:

Integration Request --> Edit --> Mapping Templates --> Content Type: text/plain --> Content Body: \
```bash
{
  "body" : "$input.body",
  "queryStringParameters" : {
      "fileName" : "$input.params('fileName')"
  }
}
```
---

### Step 7: Deploy API Gateway:

- Deploy the API to a stage (e.g., dev):
- Click on "Actions" > "Deploy API".
- Choose your stage (e.g., dev) and deploy.

---

### Step 8: Testing

- Upload a File
- Use Postman or another HTTP client:
- POST to https://.execute-api..amazonaws.com/dev/files?fileName=test.txt
- Set request body to raw and content type to text/plain.
- Enter file content (e.g., Hello World!) and send the request.

---

### Download a File

- Use Postman or another HTTP client:
- GET to https://.execute-api..amazonaws.com/dev/files?fileName=test.txt
- Verify file content is returned correctly.

---

## 💡 Credits
Project inspiration and baseline architecture guided by the tutorial from [Tech With Yeshwanth](https://www.youtube.com/@TechWithYeshwanth).
