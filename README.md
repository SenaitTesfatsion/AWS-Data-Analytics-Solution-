# Building A Data Analytics Solution In The AWS Cloud
This project shows designing an AWS architecture for data analytics infrastructure to ingest, store, and visualize clickstream data. 
In this architecture, API Gateway will be used to ingest clickstream data. The Lambda function will then transforms the data and sends it to Kinesis Data Firehose. The Firehose delivery stream is used to place all files in Amazon S3 and Amazon Athena will be used to query the files. Finally, Amazon QuickSight is used to transform data into graphics.

![DA1](https://user-images.githubusercontent.com/110143245/222317064-0a27bf88-349d-4cad-b200-6d4dd8108b21.png)

# Cloud Tools Used;
- _AWS Identity and Access Management (IAM) policy and user_
- _Amazon Simple Storage Service bucket (S3)_
- _Lambda functions_
- _AWS Kinesis Data Firehose_
- _Amazon Athena_
- _API Gateway_
- _Amazon Quicksight_


# Instructions

* [Create IAM Policy](create-iam-policy)
* [Create IAM Role and attach a policy to the role](create-iam-role-and-attach-policy-to-the-role)
* [Create an S3 bucket](create-an-s3-bucket)
* [Create a Lambda function](create-a-lambda-function)
* [Create a Kinesis Data Firehose delivery stream](create-a-kinesis-data-firehose-delivery-stream)

* [Add the firehose delivery stream ARN to the S3 bucket](add-the-firehose-delivery-stream-arn-to-the-s3-bucket)
* [Create an API in API Gateway](create-an-api-in-api-gateway)
* [Create an Athena table](create-an-athena-table)
* [Visualizing data with Quicksight](visualizing-data-with-quicksight)
* [Deleting all resources](deleting-all-resources)


# Stage 1 – Create IAM Policies 
### Stage 1A – Create IAM Policy for the Dynamodb table
- Sign in to the AWS management console
- Select the appropriate region
- Select **IAM** from the AWS services
- In the navigation pane, choose **Policies**
- Choose **Create policy** 
- In the **JSON tab**, paste the following code:
~~~
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "firehose:PutRecord",
            "Resource": "*"
        }
    ]
}
~~~
- Choose **Next: Tags** and then choose **Next: Review**

- For the policy **name**, enter "Lambda-Write-DynamoDB"

- Choose **Create policy**

_This IAM policy grants permissions to write records to Amazon Kinesis Data Firehose._


# Stage 2 - Create IAM Roles and attach a policy to it

- In the navigation pane of the **IAM** dashboard, choose **Roles**
- In the **Use case** section, on the **Use cases for other AWS service**s menu, choose **API Gateway**

- Select the **API Gateway** use case

- Choose **Next** and then choose **Next** again

- For **Role name**, enter "APIGateway-Firehose"

- Choose **Create role**

- From the **roles list**, choose the **APIGateway-Firehose role**

- In the **Permissions policies** section, on the **Add permissions** menu, choose **Attach policies**

- From the policies list, select **API-Firehose** and choose **Attach policies**

- In the **Summary** section, copy the **APIGateway-Firehose ARN** and save it for your records

 # Stage 3 - Create an S3 bucket
 - In the AWS services console choose **S3**

- Choose Create **bucket**

- For **Bucket name**, enter a unique name

- At the bottom of the page, choose **Create bucket**

- Open the bucket details by choosing the name of the bucket just created

- Choose the **Properties** tab

- In the **Bucket overview** section, copy your bucket’s Amazon Resource Name (ARN) and save it for your records


# Stage 4 - Create a Lambda function 
 - In the search box of the AWS services, enter **Lambda**
 - Choose **Create function**
 - In the filter box of the **Blueprints** section, enter ""Kinesis""
- Select the **Python 3.8** blueprint called **Process records sent to a Kinesis Firehose stream** and choose **Configure**
- For **Function name**, enter "transform-data"
- Leave everything else as default and choose **Create function**
- In the **Code** tab, replace the default code with the following:
~~~
import json
import boto3
import base64

output = []

def lambda_handler(event, context):

    for record in event['records']:
        payload = base64.b64decode(record['data']).decode('utf-8')

        row_w_newline = payload + "\n"
        row_w_newline = base64.b64encode(row_w_newline.encode('utf-8'))

        output_record = {
            'recordId': record['recordId'],
            'result': 'Ok',
            'data': row_w_newline
        }
        output.append(output_record)

    return {'records': output}
~~~

- Choose **Deploy**

- Choose the **Configuration** tab

- In the **General configuration** section, choose ****Edit** and change the **Timeout** setting to 10 seconds

- Choose **Save**

- In the **Function overview** section, copy the function ARN and save it for your records



# Stage 5 - Create a Kinesis Data Firehose delivery stream
### Stage 5A - Create the Kinesis Data Firehose delivery stream
- In AWS Management Console, search for and open the Kinesis service
- On the **Get started** card, select **Kinesis Data Firehose** and then choose **Create delivery stream**
- For **Source** and **destination**, configure these settings:
- **Source**: Direct PUT
- **Destination**: Amazon S3
- For **Transform and convert records - optional**, configure these settings:
- **Data transformation**: Enabled
- **AWS Lambda function**: ARN of the function created in Lambda
- **Version and alias**: $LATEST
- For **Destination settings**, choose **Browse**
- Select the S3 bucket created for this project and then select **Choose**
- Scroll down to the bottom of the page, choose **Create delivery stream**

### Stage 5B - Copy the ARN of the IAM role
- Open the details page for the created delivery stream 
- Choose the **Configuration** tab
- In the **Permissions** section, choose the IAM role
- Copy the **ARN of the IAM role** and save it for later use


# Stage 6 -Add the Firehose delivery stream ARN to the S3 bucket 
- Open the Amazon S3 console and open the details page for the bucket created in this project
- Choose the **Permissions** tab
- In the **Bucket policy** section, choose **Edit** and paste the following script:
~~~

{
    "Version": "2012-10-17",
    "Id": "PolicyID",
    "Statement": [
        {
            "Sid": "StmtID",
            "Effect": "Allow",
            "Principal": {
                "AWS": "<Enter the ARN for the Kinesis Firehose IAM role>"
            },
            "Action": [
                "s3:AbortMultipartUpload",
                "s3:GetBucketLocation",
                "s3:GetObject",
                "s3:ListBucket",
                "s3:ListBucketMultipartUploads",
                "s3:PutObject",
                "s3:PutObjectAcl"
            ],
            "Resource": [
                "<Enter the ARN of the S3 bucket>",
                "<Enter the ARN of the S3 bucket>/*"
            ]
        }
    ]
}
~~~
- In the script code, replace the following placeholders:
**AWS**: Paste the ARN of the Kinesis Data Firehose IAM role saved in the previous task
**Resource**: Paste the ARN of the S3 bucket for both placeholders
- **Save** your changes

# Stage 7 - Create an API in API Gateway
- Open the **API Gateway** service console
- On the **REST API** card with an open authentication, choose **Build** and configure these settings:
- **Choose the protocol**: REST
- **Create new API**: New API
- **API name**: clickstream-ingest-poc
- **Endpoint Type**: Regional
- Choose **Create API**
- In the **Resources** pane, on the **Actions** menu, choose **Create Resource**
- On the **New Child Resource** page, name the resource "poc" and choose **Create Resource**
- On the **Actions** menu, choose **Create Method**
- On the **method** menu (down arrow), choose **POST**. Choose the checkmark to save your changes
- On the **POST - Setup** page, configure the following settings:
- **Integration type:** AWS Service
- **AWS Region**: us-east-1
- **AWS Service**: Firehose
- **AWS Subdomain**: Keep empty
- **HTTP method**: POST
- **Action Type**: Use action name
- **Action**: PutRecord
- **Execution role:** Paste the ARN of the APIGateway-Firehose role created in stage 1
- **Content Handling**: Passthrough
- **Use Default Timeout**: Keep selected
- Save your changes
- Choose the **Integration Request** card
- Expand **Mapping Templates** and for **Request body passthrough**, choose **When there are no templates defined** 
- Choose **Add mapping template**
- For **Content-Type**, enter "application/json" and save your changes by choosing the check mark
- In the **Generate template** box, paste the following script:
~~~
{
    "DeliveryStreamName": "<Enter the name of your delivery stream>",
    "Record": {
        "Data": "$util.base64Encode($util.escapeJavaScript($input.json('$')).replace('\', ''))"
    }
}
~~~
- In the script code, replace the **DeliveryStreamName** placeholder value with the name of the Kinesis Data Firehose delivery stream that you created
- Choose **Save**
- Go back to the **/poc - POST - Method Execution** page

- Choose **Test**
- In the **Request Body** box, paste the following JSON:
~~~
{
"element_clicked":"entree_1",
"time_spent":67,
"source_menu":"restaurant_name",
"created_at":"2022–09–11 23:00:00"
}
~~~
- Choose **Test**

- _Review the request logs (on the right side of the window) and confirm that you see the following messages: “Successfully completed execution” and “Method completed with status: 200”._

- Replace the previous JSON by pasting the following JSON payload and then choose **Test**. Again, confirm that API Gateway processed the data successfully.
~~~
{
"element_clicked":"entree_1",
"time_spent":12,
"source_menu":"restaurant_name",
"created_at":"2022–09–11 23:00:00"
}
~~~

- _Repeat the steps for pasting the code, testing the API, and confirming success for each of the following JSON payloads:_
- Entree 4
~~~
{
"element_clicked":"entree_4",
"time_spent":32,
"source_menu":"restaurant_name",
"createdAt":"2022–09–11 23:00:00"
}
~~~
- Drink 1
~~~
{
"element_clicked":"drink_1",
"time_spent":15,
"source_menu":"restaurant_name",
"created_at":"2022–09–11 23:00:00"
}
~~~
- Drink 3
~~~
{
"element_clicked":"drink_3",
"time_spent":14,
"source_menu":"restaurant_name",
"created_at":"2022–09–11 23:00:00"
}
~~~

# Stage 8 - Create an Athena table

- Open the Athena service console

- If you are a new user: On the **Begin querying your data** card, choose **Explore the query editor**. For existing users, Athena might open the console automatically

- Choose the **Settings** tab and then choose **Manage**

- Choose **Browse** and choose the S3 bucket created in this project

- Save your actions

- Choose the **Editor** tab.
- In the **Tables and views** section, on the **Create** menu, choose **CREATE TABLE AS SELECT**
- In the **Query** editor, replace the placeholder code with the following script:
~~~
CREATE EXTERNAL TABLE my_ingested_data (
element_clicked STRING,
time_spent INT,
source_menu STRING,
created_at STRING
)
PARTITIONED BY (
datehour STRING
)
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
with serdeproperties ( 'paths'='element_clicked, time_spent, source_menu, created_at' )
LOCATION "s3://<Enter your Amazon S3 bucket name>/"
TBLPROPERTIES (
"projection.enabled" = "true",
"projection.datehour.type" = "date",
"projection.datehour.format" = "yyyy/MM/dd/HH",
"projection.datehour.range" = "2021/01/01/00,NOW",
"projection.datehour.interval" = "1",
"projection.datehour.interval.unit" = "HOURS",
"storage.location.template" = "s3://<Enter your Amazon S3 bucket name>/${datehour}/"
)
~~~
- In the script you pasted, replace the following placeholder values:
**LOCATION**: Replace <Enter your Amazon S3 bucket name> with the name of your bucket
**storage.location.template**: Replace <Enter your Amazon S3 bucket name> with the name of your bucket
- Choose **Run**
- The query creates the **my_ingested_data** table
- Create a new query by choosing the plus sign (+) (at the top-right of the query editor)
- In the query editor, paste **SELECT * FROM - my_ingested_data**; and choose **Run**
- _The query should produce results with the entries that you ran in API Gateway._

# Stage 9 - Visualizing data with QuickSight
- Open the QuickSight service console

- Choose **Sign up for QuickSight**

- In the upper-right corner, **choose Standard**

- Enter a unique account name and your email address

- Allow access to the S3 bucket created in this project

- Choose **Finish**

- After you sign up for Amazon QuickSight, go to the QuickSight console

- Open the user menu by choosing the user icon and then choose **Manage QuickSight**

- In the navigation pane, choose **Security & permissions** and in **QuickSight access to AWS services**, choose **Manage**

- Under **Amazon S3**, choose **Select S3 buckets**

- Select the bucket created in this project, and also select **Write permission for Athena Workgroup**

- Choose **Finish** and save your changes

- Return to the QuickSight console

- In the **Analyses** tab, choose **New analysis**

- Choose **New dataset**

- Choose **Athena** and configure the following settings:
- **Name datasource**: poc-clickstream
- **Select workgroup**: [primary]
- Choose **Create data source**
- In the **Choose your table** dialog box, select the **my_ingested_data** table, and choose **Select**
- In the **Finish dataset creation** dialog box, make sure that **Import to SPICE for quicker analytics** is selected, and choose **Visualize**
- View your visualization results by selecting **field items** and **visual types** for the diagram

# Stage 10 - Delete all resources
### A. Delete the QuickSight dashboards
- Return to the QuickSight console by choosing the **QuickSight** icon (in the upper-left area of the webpage)
- Choose **Analyses**
- On the **my_ingested_data** card, open the actions menu by choosing the ellipsis icon
- On the actions menu, choose **Delete** and confirm your action
- Confirm your action
- In the navigation pane, choose **Datasets**
- For each dataset, on the actions menu, choose **Delete** and confirm your action
### B. Delete your QuickSight account
_If you will not be using QuickSight in the future, you can delete your account._
- In the QuickSight console, choose the **user icon** and then choose **Manage QuickSight**
- In the navigation pane, choose **Account settings**
- Choose **Delete account** and confirm your action
### C. Delete the S3 bucket
- Return to the Amazon S3 console
- Choose the bucket created for this project and delete all files in the bucket
- Select the bucket name, choose **Delete**, and confirm your action
### D. Delete the Athena table and queries
- Return to the Athena console
- Make sure that you are on the **Editor** tab
- In the navigation pane, on the actions menu for **my_ingested_data**, choose **Delete table** and confirm your action
- Close the queries that you created
- Choose the **Settings** tab and then choose **Manage**
- Remove the path to the S3 bucket and save your changes
### E. Delete the API Gateway configuration
- Return to the API Gateway console
- Select the API that you created
- On the **Actions** menu, choose **Delete** and confirm your action
### F. Delete the Kinesis Data Firehose delivery stream.
- Return to the Kinesis console
- In the navigation pane, choose **Delivery streams**
- Select the delivery stream that you created, choose **Delete**, and confirm your action
### G. Delete the Lambda function
- Return to the Lambda console
- Select the **transform-data** Lambda function
- On the **Actions** menu, choose **Delete**, and confirm your action
### 7. Delete the IAM roles and policies
- Return to the IAM dashboard
- In the navigation pane, choose **Roles**
- Select **APIGateway-Firehose**, choose **Delete**, and confirm your action
- In the navigation pane, choose **Policies**
- Select **API-Firehose**
- On the **Actions** menu, choose **Delete** and confirm the deletion



































