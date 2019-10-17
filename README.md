# Build a conversational chatbot to gain business insights
Conversational interfaces are transforming the way people interact with software applications and services. Increasingly, people are opting to interact with a bot when they need an answer to a question, to set a reminder, or to obtain a product or service.

With Amazon Lex, you can bring this same level of convenience to data. By allowing users to explore datasets by asking a series of questions, and maintaining a conversational context, you can provide a whole new experience and relationship with data.

## Introduction

In this lab, you will learn how to use Amazon Lex to implement a business intelligence (BI) chatbot, which we refer to as “BIBot”, although you can customize it to use a different name. BIBot can respond to user questions about data in a database, by converting the questions into backend database queries, and transforming the result sets into natural language responses. For example, the request “tell me the increase in inventory last month” could be translated to “select sum(item_qty) from inventory where month(received_date) = 10”.

## Step 0: Create development environment using AWS Cloud9

1. Login to your AWS Account and select **us-west-2** region. This is the region where you will do rest of the labs.
2. Create Cloud9 environment :
	1.  Go to the Cloud9 console.
	2. Choose the **Create environment** button.
	3. On the  **Name environment**  page, for  **Name**, enter `BuilderSession`.
	4. (Optional) To add a description to your environment, enter it in  **Description**.
	5. Choose  **Next step**.
	6. On the **Configure settings** page, for **Environment type**, choose **Create a new instance for environment (EC2)**.
	7. Leave rest of the values to default.
	8. Choose **Next step**.
	9. On the **Review** page, choose **Create environment**. Wait while AWS Cloud9 creates your environment. This can take several minutes.
	10. After AWS Cloud9 creates your environment, it displays the AWS Cloud9 IDE for the environment. 
3. Clone lab github repo in Cloud9 IDE:
	1. Go to Cloud9 environment you just created.
	2. On the bottom half of your IDE, you will see a termial window. Feel free to resize that window as you feel appropriate. Run following command to clone lab github repo.
	 `git clone https://github.com/sshvans/amazon-lex-bi-bot.git`

### Summary
You have successfully setup a a cloud IDE and cloned the lab github repository.

## Step 1: Create sample database

**Note:** *Replace YOUR_INITIALS with initials of your name.*

1. Go to AWS S3 console and create a bucket with name `YOUR_INITIALS-bibot-tickit-data`. This is the bucket where you will store a copy of the TICKIT sample data.
2. Create another S3 bucket called `YOUR_INITIALS-bibot-db-output`. This S3 bucket will be used for Athena to store output from queries.
3. Set environment variables:
	1. Go to your Cloud9 IDE, open file *export-env.sh*.
	2.  Provide the value for your Athena S3 bucket you created above. Replace `REPLACE-ME-ATHENA-BUCKET` with the bucket name `YOUR_INITIALS-bibot-tickit-data`
	3. **Save** the file.
	4. In terminal window, run the following command to set the updated values.
		```
		cd amazon-lex-bi-bot
		source export-env.sh
		```
4. In your Cloud9 IDE terminal window, run the following commands to copy TICKIT sample data files to your S3 bucket and build TICKIT database.
	```
	bash setup-db.sh
	```
### Summary
You have successfully created a sample TICKIT database which will be used by your BIBot.

## Step 2: Create IAM role

**Note:**  *Replace YOUR_INITIALS with initials of your name.*

1. From your Cloud9 IDE terminal, run following command to launch a cloudformation stack, which will create a Lambda execution IAM role.
`aws cloudformation create-stack --stack-name YOUR_INITIALS-bbot-iam-roles --template-body file://iam-roles.yaml --region us-west-2 --capabilities CAPABILITY_IAM`
2. Make a note of the IAM role ARN:
	1. In a new tab, open the AWS CloudFormation console.
	2. From the list of stack, click the CloudFormation stack named **YOUR_INITIALS-bbot-iam-roles**. If the stack is still in-progress, wait for it to complete.
	3. Click **Outputs** tab.
	4. Make a note of the **IAMRoleArn** value.

### Summary
You have created an IAM role which you will use with the Lambda function to give it a permission to perform actions on Amazon Athena, Amazon S3, and Amazon Lex.

## Step 3: Create Lambda functions

You will use lambda functions which will fulfill the requests from Amazon Lex. It will perform queries on the Athena TICKIT sample database, and send response back to Amazon Lex.

We have created a script to expedite the process of creating Lambda function. Script need a *Lambda IAM role ARN* environment variable, to run successfully.

To set environment variable:
1. In your Cloud9 IDE, open file *export-env.sh*.
2.  Provide the value for LAMBDA_ROLE_ARN. Replace `REPLACE-ME-LAMBDA-IAM-ROLE-ARN` with the IAM role ARN you copied in Step 2 above.
3. **Save** the file.
4. In terminal window, run the following command to set the updated values for the LAMBDA_ROLE_ARN environment variable.
`source export-env.sh`

To create Lambda functions:
1. In your terminal window run the following commands.
	```
	cd lambda
	bash ../create-lambda.sh
	```

## Recap

So far, you have successfully created:
- An Athena database with sample TICKIT database.
- An IAM role for the Lambda functions which gives permission to perform queries on Athena database and store its output on S3.
- Lambda functions for each intent

## Step 4: Create Amazon Lex Bot

1.  Sign in to the AWS Management Console and open the Amazon Lex console at [https://console.aws.amazon.com/lex/](https://console.aws.amazon.com/lex/).
2. Create a bot.
3.  If you are creating your first bot, choose **Get Started**. Otherwise, choose **Bots**, and then choose **Create**.
4.  On the **Create your Lex bot** page, choose **Custom bot** and provide the following information:
	-  **App name**: BIBot
	-  **Output voice**: Matthew
	-  **Session timeout** : 5 minutes.
	-  **COPPA**: Choose No.
5.  Choose **Create**.

## Step 5: Create Intent

Now, create the `Count_Intent` intent , an action that the user wants to perform, with the minimum information needed. You add slot types for the intent and then configure the intent later.  
  
To create an intent:  

1.  In the Amazon Lex console, choose the plus sign (+) next to **Intents**, and then choose **Create intent**.
2.  In the **Create intent** dialog box, type the name of the intent (Count_Intent), and then choose **Add**.

## Step 6: Create Slot types

Create the slot types, or parameter values, that the `Count_Intent` intent uses.  

To create slot types:  

1.  In the left menu, choose the plus sign (+) next to **Slot types.** Choose **Create slot type.**
2.  In the **Add slot type** dialog box, add the following:
	-  **Slot type name** – cat_desc
	-  **Description** – Categories of events in the TICKIT database
	-  Choose **Expand Values.**
	-  **Value** – Type following values one at a time, and click **+** symbol to add.
		-  Musical theatre
		-  All opera and light opera
		-  All non-musical theatre
		-  All rock and pop music concerts
3.  You dialog should look like below: add image
4.  Choose **Add slot to intent**.
5.  On the **Intent** page, keep **Required** checkbox **unchecked**. Change the name of the slot from **`slotOne`** to **`cat_desc`**. Change the prompt to **`The type of event`**
6.  Repeat Step 1 through Step 4 using the values in the following table:

|  Name | Description | Values | Slot name | Prompt
|--|--|--|--|--|
|event_name|Name of events in TICKIT database|Joshua Radin, Rhinoceros|event_name|The name of the event|

7. Add other slots to the intent, using the top row and values in the following table:

| Name | Slot type | Prompt|
|--|--|--|
|event_month|AMAZON.Month|The month of the event|
|venue_city|AMAZON.US_CITY|The city where the event takes place|
|venue_state|AMAZON.US_STATE|The state where the event takes place|
|venue_name|AMAZON.MusicVenue|The venue where the event takes place|

## Step 7: Configure the Intent

Configure the `Count_Intent` intent to fulfill a user's request.  
  
To configure the intent, on the `Content_Intent` configuration page, configure the intent as follows:  

1.  **Sample utterances** – Type the following strings. The curly braces {} enclose slot names.
	-  How many tickets were sold
	-  Ticket sales for {event_name} in {venue_state}
	-  Count of tickets sold in {event_month} in {venue_city}
	-  Count of tickets sold in {venue_city} for {event_name}
	-  Tickets sold in {venue_state} in {event_month}
	-  Tickets sold in {venue_city} {venue_state} for {event_name}
	-  How many tickets were sold at {venue_name} in {event_month}
	-  How many tickets were sold for {cat_desc}
2.  **Lambda initialization and validation** – Leave the default setting.
3.  **Confirmation prompt** – Leave the default setting.
4.  **Fulfillment** – Perform the following tasks:
	1.  Choose **AWS Lambda function**.
	2.  Choose `**BIBot_Count_Intent**`.
	3.  If the **Add permission to Lambda function** dialog box is shown, choose **OK** to give the `Count_Intent` intent permission to call the `BIBot_Count_Intent` Lambda function.
	4.  Leave **None** selected.
5.  **Response** - Perform the following tasks:
	1.  Click **+ Add Message**
	2.  Type `via code hook`, and press **Enter.**
6.  Click **Save Intent**

## Step 8: Configure the Bot

Configure error handling for the `BIBot` bot.  

1.  Navigate to the `BIBot` bot. Choose **Editor**. and then choose **Error Handling**.
2.  Use the **Editor** tab to configure bot error handling.
	-  In **Clarification Prompts** text box, enter `Come again?` and click **+** symbol**.**
3.  Click **Save.**

## Step 9: Build and test the bot

1.  To build the `BIBot` bot, choose **Build**. It can take some time to build.
2.  To test the bot, in the **Test Bot** window, start communicating with your Amazon Lex bot. You can use any of the sample utterances to communicate with your bot. For eg.
	-  How many tickets were sold
	-  How many tickets were sold in boston?
	-  How many in chicago?

## Step 10: Create rest of the Intents

Go to your Cloud9 IDE and in the terminal window, run the following commands to create remaining Intents.
```
bash delete.sh  
bash build-db.sh  
bash build-bot.sh
```

## Step 11: Test BI bot

To query 