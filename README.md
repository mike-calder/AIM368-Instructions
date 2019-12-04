# Sample Collection (Lab 1)

Our goal in this section is to gather data from our Fulfillment Center (FC) and store it as a dataset that can later be used to train machine learning models.

## Create an S3 bucket to store the samples in

Let's work backwards by first creating a place to store our data. We will use S3 as our datastore because it is well-equipped to handle very large datasets. Later in the workshop we will discover that SageMaker is well integrated with S3, making it easy to train ML models.

<details><summary>Instructions</summary>

1. Navigate to **S3** from the AWS home page.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/785900f1b98f44a78ece6ba902080043/v1/2-1-Go-To-S3.png)
2. Select the **+ Create bucket** button in the top-left.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/785900f1b98f44a78ece6ba902080043/v1/2-2-S3-Home.png)
3. Type in `aim368-samples-bucket-<FIRSTNAME>-<LASTNAME>` (e.g. `aim368-samples-bucket-mike-calder`)
> :warning: **Please follow all naming instructions**: Failure to do so could prevent parts of the lab from working!
4. Click the **Create** button in the bottom-left of the pane.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/785900f1b98f44a78ece6ba902080043/v1/2-3-Create-Bucket.png)
5. Verify the bucket was created by finding it in the list.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/785900f1b98f44a78ece6ba902080043/v1/2-4-Bucket-Created.png)

</details>

## Create a Kinesis Firehose to automatically batch the samples

We need to get our data from the FC to our new S3 Bucket, but the FC is publishing the data on a sample by sample basis. It would not make sense to create a new S3 file (or update a file) for every single sample, so we need an aggregation step. To accomplish this, we can create a Kinesis Firehose that automatically aggregates our data into batches and only writes to S3 after the batch size is reached or after a set amount of time.

<details><summary>Instructions</summary>

1. Navigate to **Kinesis** from the AWS home page.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/785900f1b98f44a78ece6ba902080043/v1/3-1-Go-To-Kinesis.png)
2. Select the **Get started** button in the top-middle area.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/785900f1b98f44a78ece6ba902080043/v1/3-2-Kinesis-Home.png)
3. In the Kinesis Firehose box, click **Create delivery stream**.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/785900f1b98f44a78ece6ba902080043/v1/3-3-Create-Firehose-1.png)
4. Enter `SampleCollectionFirehose` for the delivery stream name.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/785900f1b98f44a78ece6ba902080043/v1/3-4-Create-Firehose-2.png)
5. Click the **Next** button in the bottom-right.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/785900f1b98f44a78ece6ba902080043/v1/3-5-Create-Firehose-3.png)
6. Click the **Next** button in the bottom-right again.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/785900f1b98f44a78ece6ba902080043/v1/3-6-Create-Firehose-4.png)
7. Under **S3 destination**, select your bucket and click **Next** again.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/785900f1b98f44a78ece6ba902080043/v1/3-7-Create-Firehose-5.png)
8. Under **Permissions** click **Create new or choose**.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/785900f1b98f44a78ece6ba902080043/v1/3-8-Create-Firehose-6.png)
9. Select **SampleCollectionFirehoseRole** for the IAM Role.
10. Select **SampleCollectionFirehosePolicy** for the Policy Name.
11. Click the **Allow** button in the bottom-right.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/785900f1b98f44a78ece6ba902080043/v1/3-9-Create-Firehose-7.png)
12. Click the **Next** button in the bottom-right again.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/785900f1b98f44a78ece6ba902080043/v1/3-10-Create-Firehose-8.png)
13. Select **Create delivery stream** in the bottom-right.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/785900f1b98f44a78ece6ba902080043/v1/3-11-Create-Firehose-9.png)
14. Verify the Kinesis Firehose has started creating.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/785900f1b98f44a78ece6ba902080043/v1/3-12-Firehose-Creating.png)
16. Verify the Kinesis Firehose was created.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/785900f1b98f44a78ece6ba902080043/v1/3-13-Firehose-Created.png)

</details>

## Create a Lambda function to connect the SNS topic to the Firehose

Finally, we need to add a serverless compute step to read the data notifications from the FC and write the data to the Kinesis Firehose. We will use a Lambda with an SNS trigger to accomplish this.

<details><summary>Instructions</summary>

1. Navigate to **Lambda** from the AWS home page.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/785900f1b98f44a78ece6ba902080043/v1/4-1-Go-To-Lambda.png)
2. Select the **Create function** button in the top-right.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/785900f1b98f44a78ece6ba902080043/v1/4-2-Create-Function-1.png)
3. Enter `SampleCollectionLambdaFunction` for the function name.
4. Select **Python 3.8** in the dropdown for the runtime.
5. Click **Choose or create an execution role**.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/785900f1b98f44a78ece6ba902080043/v1/4-3-Create-Function-2.png)
6. Click the **Use an existing role** radio button.
7. Select **SampleCollectionLambdaRole** in the dropdown.
8. Click the **Create function** button in the bottom-right.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/785900f1b98f44a78ece6ba902080043/v1/4-4-Create-Function-3.png)
9. Verify the function has been created.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/785900f1b98f44a78ece6ba902080043/v1/4-5-Function-Created.png)
10. Copy-paste the following code into the **Function code** box: (replacing the sample code)
    ```python
    import boto3
    
    firehose = boto3.client('firehose')
    
    def lambda_handler(event, context):
        sample = event['Records'][0]['Sns']['Message']
        firehose.put_record(
            DeliveryStreamName = 'SampleCollectionFirehose',
            Record = {'Data': sample.encode('UTF-8')}
        )
    ```
11. Click the **Save** button in the top-right corner.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/785900f1b98f44a78ece6ba902080043/v1/4-6-Function-Code.png)
12. In the top-left, click the **+ Add trigger** button.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/785900f1b98f44a78ece6ba902080043/v1/4-7-Add-Trigger-1.png)
13. Select **SNS** in the trigger configuration dropdown.
14. Click the **Add** button in the bottom-right.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/785900f1b98f44a78ece6ba902080043/v1/4-8-Add-Trigger-2.png)
15. Verify the trigger has been added.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/785900f1b98f44a78ece6ba902080043/v1/4-9-Trigger-Added.png)

</details>

# Model Training (Lab 2)

In this module, we will start by loading our data into a SageMaker Notebook for analysis. Then we will train ML models, choose the best model, and finally deploy it, all using SageMaker managed hardware.

## Create a SageMaker Notebook

SageMaker Notebooks provides familiar tooling via Jupyter Notebooks running on cloud hardware. Let's first set up a notebook so we can import our data and train some ML models.

<details><summary>Instructions</summary>

1. Navigate to **Amazon SageMaker** from the AWS home page.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/dc759d1485734b1481cbd9beab219de2/v1/6-1-Go-To-SageMaker.png)
2. Select **Notebook Instances** in the left panel under **Notebooks**.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/dc759d1485734b1481cbd9beab219de2/v1/6-2-SageMaker-Home.png)
3. Click the **Create notebook instance** button in the top-right.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/dc759d1485734b1481cbd9beab219de2/v1/6-3-Create-Notebook-1.png)
4. Enter `ModelTrainingNotebook` for the notebook instance name.
5. Select **ml.c5.2xlarge** for the notebook instance type.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/dc759d1485734b1481cbd9beab219de2/v1/6-4-Create-Notebook-2.png)
5. Click the **Create notebook instance** button in the bottom-right.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/dc759d1485734b1481cbd9beab219de2/v1/6-5-Create-Notebook-3.png)
6. Verify the Amazon SageMaker notebook has started creating.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/dc759d1485734b1481cbd9beab219de2/v1/6-6-Notebook-Creating.png)
7. Wait for the notebook to have the **InService** status. (3-5 minutes)
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/dc759d1485734b1481cbd9beab219de2/v1/6-7-Notebook-Created.png)

</details>

## Train models in the SageMaker Notebook

Now that the SageMaker Notebook is set up, we can import our data and leverage built-in SageMaker training algorithms to generate ML models. No python or data science experience necessary, just follow the steps to import our notebook files. These files will walk you through some basic data analytics and help you train ML models with three different algorithms.

<details><summary>Instructions</summary>

1. Click **Open JupyterLab** in the right-most column.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/dc759d1485734b1481cbd9beab219de2/v1/6-8-Open-JupyterLab.png)
2. Select the git clone logo in the top-right of the middle pane.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/dc759d1485734b1481cbd9beab219de2/v1/6-9-Clone-Repo-1.png)
3. Enter `https://github.com/mike-calder/AIM368-Notebooks.git` and hit **CLONE**.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/dc759d1485734b1481cbd9beab219de2/v1/6-10-Clone-Repo-2.png)
> :warning: There will be no visual indication that the clone is progressing, but it should complete within a minute.
4. Double-click the **AIM368-Notebooks** folder in the top-left.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/dc759d1485734b1481cbd9beab219de2/v1/6-11-Open-Notebook-1.png)
5. Double-click the **Data-Analysis.ipynb** file in the top-left.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/dc759d1485734b1481cbd9beab219de2/v1/6-12-Open-Notebook-2.png)
6. Walk through the notebook by typing *Shift+Enter* on each individual cell.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/dc759d1485734b1481cbd9beab219de2/v1/6-13-Open-Notebook-3.png)
7. Repeat this process for each of the three model training notebooks:
    * **K-Nearest-Neighbors.ipynb**
    * **Linear-Learner.ipynb**
    * **XGBoost.ipynb**

    ![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/dc759d1485734b1481cbd9beab219de2/v1/6-14-Open-Notebook-4.png)

</details>

## Deploy a model from the SageMaker Notebook

Now that we have trained some models, how do we use them to predict the duration of future human-robot interactions? For that, we need to deploy a model. Luckily, all we need is one line of code and we can deploy our model to a SageMaker managed cloud instance that's ready to recieve inference requests.

<details><summary>Instructions</summary>

1. Of the three models you trained, choose one to deploy for the accuracy competition.
2. Below that model's training output, create a new notebook cell with the following code:

    ```python
    model.deploy(initial_instance_count = 1,
                 instance_type = 'ml.c5.2xlarge',
                 endpoint_name = 'LiveInferenceEndpoint')
    ```
3. Deploy the model by pressing *Shift+Enter* in the new cell.

</details>

# Live Inference (Lab 3)

In this module we will create a live inference API that routes requests to our SageMaker endpoint. This will allow our FC edge compute resources to call the ML model and receive duration estimates for upcoming human-robot interactions.

## Create a Lambda function to handle API Gateway requests

Once again let's work backwards by first creating a Lambda to call our SageMaker endpoint. In a production situation, this would allow us to test our model integration before exposing it to traffic from the FC.

<details><summary>Instructions</summary>

1. Navigate to **Lambda** from the AWS home page.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/081d07a12a994c7f936e1eca52901f18/v1/7-1-Go-To-Lambda.png)
2. Select the **Create function** button in the top-right.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/081d07a12a994c7f936e1eca52901f18/v1/7-2-Create-Function-1.png)
3. Enter `LiveInferenceLambdaFunction` for the function name.
4. Select **Python 3.8** in the dropdown for the runtime.
5. Click **Choose or create an execution role**.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/081d07a12a994c7f936e1eca52901f18/v1/7-3-Create-Function-2.png)
6. Click the **Use an existing role** radio button.
7. Select **LiveInferenceLambdaRole** in the dropdown.
8. Click the **Create function** button in the bottom-right.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/081d07a12a994c7f936e1eca52901f18/v1/7-4-Create-Function-3.png)
9. Verify the function has been created.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/081d07a12a994c7f936e1eca52901f18/v1/7-5-Function-Created.png)
10. Copy-paste the following code into the **Function code** box: (replacing the sample code)
    ```python
    import boto3
    
    sagemaker = boto3.client('sagemaker-runtime')
    
    def lambda_handler(event, context):
        features = event['queryStringParameters']['features']
        response = sagemaker.invoke_endpoint(EndpointName = 'LiveInferenceEndpoint',
                                             ContentType = 'text/csv',
                                             Body = features)
        
        inference = response['Body'].read().decode()
        
        return {
            'statusCode': 200,
            'body': inference
        }
    ```
11. Click the **Save** button in the top-right corner.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/081d07a12a994c7f936e1eca52901f18/v1/7-6-Function-Code.png)
12. Verify that the function updated successfully.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/081d07a12a994c7f936e1eca52901f18/v1/7-7-Function-Updated.png)

</details>

## Create an API Gateway endpoint that can receive inference requests

Now that we have a Lambda capable of calling our ML model, we need to provide a way for the FC to trigger the lambda with new data. API Gateway will allow us to set up an external API that recieves data and routes it to the Lambda we created. Then the Lambda can call the ML model and return the estimated duration back to the caller.

<details><summary>Instructions</summary>

1. Navigate to **API Gateway** from the AWS home page.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/081d07a12a994c7f936e1eca52901f18/v1/8-1-Go-To-API-Gateway.png)
2. Click **Get Started** in the top-middle, then **Ok** to clear the popup.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/081d07a12a994c7f936e1eca52901f18/v1/8-2-API-Gateway-Home.png)
3. Choose **REST** for the protocol type and select the **New API** radio button.
4. Type `LiveInferenceAPI` for the name, then click **Create API** in the bottom-right.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/081d07a12a994c7f936e1eca52901f18/v1/8-3-Create-API.png)
5. At the top of the page, click the **Actions** drop down and select **Create Resource**.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/081d07a12a994c7f936e1eca52901f18/v1/8-4-Create-Resource-1.png)
6. Type `liveinference` as the resource name, then click **Create Resource**.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/081d07a12a994c7f936e1eca52901f18/v1/8-5-Create-Resource-2.png)
7. At the top of the page, click the **Actions** drop down and select **Create Method**.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/081d07a12a994c7f936e1eca52901f18/v1/8-6-Create-Method-1.png)
8. Click the drop down that appears, select **GET**, then click the check mark to confirm.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/081d07a12a994c7f936e1eca52901f18/v1/8-7-Create-Method-2.png)
9. On the page that appears, check the checkbox to enable **Use Lambda Proxy integration**.
10. In the **Lambda Function** field, type `LiveInferenceLambdaFunction` and click **Save**.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/081d07a12a994c7f936e1eca52901f18/v1/8-8-Create-Method-3.png)
11. Click **Ok** to confirm the new permissions for your Lambda function.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/081d07a12a994c7f936e1eca52901f18/v1/8-9-Create-Method-4.png)
12. After a few moments you should see a diagram of your **GET - Method Execution**, click **Method Request**.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/081d07a12a994c7f936e1eca52901f18/v1/8-10-Query-Parameters-1.png)
13. Click on **URL Query String Parameters** to expand the section and then **+ Add query string**.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/081d07a12a994c7f936e1eca52901f18/v1/8-11-Query-Parameters-2.png)
14. Type `features` in the box that says *myQueryString*, then click the check mark.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/081d07a12a994c7f936e1eca52901f18/v1/8-12-Query-Parameters-3.png)
15. At the top of the page, click the **Actions** drop down and select **Deploy API**.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/081d07a12a994c7f936e1eca52901f18/v1/8-13-Deploy-API-1.png)
16. For **Deployment Stage** select **\[New Stage\]** to expand the section.
17. Type in `<FIRSTNAME>-<LASTNAME>` (e.g. `mike-calder`) for the stage name and click **Deploy**.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/081d07a12a994c7f936e1eca52901f18/v1/8-14-Deploy-API-2.png)
18. In the middle pane, click the arrow to expand your stage view.
19. Under your **liveinference** resource, click **GET** to open the method page.
20. In the right pane, copy the **Invoke URL** at the top. You will need this for the next section.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/081d07a12a994c7f936e1eca52901f18/v1/8-15-Invoke-URL.png)
> :warning: Make sure this URL ends in **liveinference**, if it does not please ask for help!

</details>

## Update the FC edge compute Lambda to call your endpoint

Next, we need to provide your API Gateway endpoint to the FC edge compute so it can begin to send inference requests. In this lab, the FC edge compute is simulated by a Lambda function. Follow the steps below to point that Lambda function at your API Gateway. Once completed, the simulated FC will start requesting inferences from your deployed model about 5 times per second.

<details><summary>Instructions</summary>

1. Navigate to **Lambda** from the AWS home page.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/081d07a12a994c7f936e1eca52901f18/v1/9-1-Go-To-Lambda.png)
2. Select **EdgeComputeLambdaFunction**.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/081d07a12a994c7f936e1eca52901f18/v1/9-2-Lambda-Functions.png)
3. Scroll down to **Environment Variables**.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/081d07a12a994c7f936e1eca52901f18/v1/9-3-Environment-Variable-1.png)
4. For the **LIVE_INFERENCE_API_GATEWAY_URL**, paste your URL from the previous section.
5. Click the **Save** button in the top-right.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/081d07a12a994c7f936e1eca52901f18/v1/9-4-Environment-Variable-2.png)
6. Verify the function updated successfully. You should appear on the leaderboard within a couple of minutes.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/081d07a12a994c7f936e1eca52901f18/v1/9-5-Environment-Variable-3.png)

</details>

# Automated Retraining

This section will be used as a starting point for an automated retraining discussion. There are many different strategies in this space, but almost all of them begin with some form of performance monitoring.

## Monitor performance with CloudWatch dashboards

When facing real world problems, it's important that the resulting solutions include a feedback loop via metrics and alarms. This will allow you to monitor your solution's performance and react to any errors or regressions. In this lab, we have set up a very simple dashboard to illustrate how CloudWatch can help fill this role.

<details><summary>Instructions</summary>

1. Navigate to **CloudWatch** from the AWS home page.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/75e7b2a625d64da9ac4e26806521463f/v1/10-1-Go-To-CloudWatch.png)
2. Select **Dashboards** in the top of the left pane.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/75e7b2a625d64da9ac4e26806521463f/v1/10-2-CloudWatch-Home.png)
3. Click on **LiveInferenceDashboard** in the list.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/75e7b2a625d64da9ac4e26806521463f/v1/10-3-Dashboards.png)
4. Monitor the performance of your ML application.
![](https://ee-assets-prod-us-east-1.s3.amazonaws.com/modules/75e7b2a625d64da9ac4e26806521463f/v1/10-4-Dashboard.png)

</details>
