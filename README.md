# Tech Girls are Superheroes

Workshop in Sydney to learn how to make a chatbot on 15 June 2019.

You're going to set up two cloud services: AWS Lex for the bot itself, and AWS Lambda for all of the functions and logic to retrieve information from the back-end software. That's it!

## Getting Started

Open up a web browser and go to [http://bit.ly/tgas-syd](http://bit.ly/tgas-syd). Login using your group number and the password `Rubrik123!`

For example, if you are in group 4 then your username is group4 and the password is `Rubrik123!`

# AWS Lambda

AWS Lambda is a compute service that lets you run code without provisioning or managing servers. AWS Lambda executes your code only when needed and scales automatically, from a few requests per day to thousands per second. You pay only for the compute time you consume - there is no charge when your code is not running. With AWS Lambda, you can run code for virtually any type of application or backend service - all with zero administration. All you need to do is supply your code in one of the languages that AWS Lambda supports (currently Node.js, Java, C#, Go and Python).

For this guide, we're going to give Roxie the power to get status on our Rubrik cluster that lives on-premises. This is driven by a lambda function written for the Python 2.7 runtime and is available in the Roxie repository on GitHub. Once you get comfortable with the process, feel free to add more functions to your bot to meet your unique use cases.

```
Note: We'd also sincerely love it if you would contribute any interesting functions you create back into the Roxie repository on GitHub. Share the love!
```

Let's start by building the function used to get cluster status.

## Function Creation

Start by navigating to the AWS Lambda service and clicking the **Create Function** button and choosing the Author From Scratch option.

*   Name: `get_cluster_status`
*   Runtime: Python 2.7
*   Role: Create a new role from one or more templates
*   Role Name: `roxie_lambda`
*   Policy Templates: Leave blank

Click on the **Create Function** button at the bottom right corner to proceed. The creation process may take a little while to complete.

Once finished, you will be taken to the `get_cluster_status` function page. There's a lot of menu items on this page - don't be alarmed! We're going to come back to this page in a moment.

## Modifying the Lambda Role

Remember the `lambda_vpc` IAM Policy that was generated earlier in this guide? It's time to use that policy to add more permissions to the brand new `roxie_lambda` role that you just created during the creation of your lambda function.

```
Note: Skipping this step will result in not being able to choose a VPC for your Lambda function's network. You will be limited to communicating with devices on the public Internet.
```

Navigate back to IAM > Roles and click on your `roxie_lambda` role to edit it. In the next screen, click on the large blue **Attach Policies** button.

![Roxie Lambda Role Summary](/docs/images/roxie-lambda-summary-1.jpg)

Next, select the `lambda_vpc` policy and click on the **Attach Policy** button in the bottom right corner. You should now have two policies attached to the `roxie_lambda` role:

*   `AWSLambdaBasicExecutionRole-{guid}`
*   `lambda_vpc`

![Roxie Lambda Role Summary](/docs/images/roxie-lambda-summary-2.jpg)

The role for your Lambda function is now able to use VPC networking, which results in being able to route back to your on-premises environment.

```
Note: If desired, this is also a good time to add a description to your role since it defaults to a blank value. A description is very helpful to you or your team! Find the Role Description field and click the Edit link next to it. For this example, we're going to use Role for Lambda functions used with Roxie.
```

You can now navigate back to the `get_cluster_status` Lambda function to proceed with configuration. There is no more work required in IAM.

## Function Configuration

There are only three sections to edit in the Lambda function page:

1.  [Function Code](#function-code): This is where the Python code for getting cluster status is loaded.
1.  [Environmental Variables](#environmental-variables): This is where the connection information for your Rubrik cluster is stored.
1.  [Network](#network): This is where the VPC network is selected.

### Function Code

For this guide, we're going to use the contents of the `get_cluster_status.py` file found in the Roxie repository on GitHub. This is a very safe command that only retrieves the status of the nodes in your cluster. It's a good use case for your first time using Roxie.

Locate the Function Code window and delete any code that already exists in the lambda_function window. Then, paste the contents of the `get_cluster_status.py` file into the Function Code window. The resulting output should look like the image below:

![Function Code for get_cluster_status](/docs/images/get-cluster-status-function-code.jpg)

You can now click the **Save** button in the upper right corner of the window to save your progress. It should change from orange to a greyed-out color upon success.

### Environmental Variables

Next, the function needs to know how to connect to your cluster. We feel it's better to keep static information out of the function code. 

For this step, it's important to add two environment variables: `CLUSTER_IP` and `AUTH_TOKEN`. This can be done by scrolling down below the Function Code area and locating the Environmental Variables area.

1.  `CLUSTER_IP`: Enter the IP address of a node in your cluster. Make sure it is reachable from the VPC network that you plan to use for Roxie. This will be covered in the next section.
1.  `AUTH_TOKEN`: Enter the token created in the Generate a Rubrik API Token section.

![Environmental Variables for get_cluster_status](/docs/images/environmental-variables.jpg)

With those values updated, you can now click the **Save** button in the upper right corner of the window to save your progress. It should change from orange to a greyed-out color upon success.

### Network

Now that the function code is loaded, it's time to choose a VPC for your Lambda function to use from a networking perspective. Scroll down to the Network section and load your specific VPC information.

As an example, the Rubrik Build lab uses a specific OR Demo VPC to communicate to the on-premises Rubrik cluster. The image below shows what a successful network configuration might look like for you:

![Network Settings for get_cluster_status](/docs/images/get-cluster-status-network.jpg)

Once finished, you can now click the **Save** button in the upper right corner of the window to save your progress. It should change from orange to a greyed-out color upon success.

```
Note: If the save button remains orange, it typically means that you have not properly added the EC2 permissions necessary for the Lambda role to attach to your VPC network.
```

## Function Testing

Before proceeding further, it's a good idea to test your Lambda function to make sure that the code and networking configuration are functioning properly. To do this, use the built in Test functionality in the function.

Click the **Test** button in the top right corner. Next, make sure the **Create New Test Event** radio button is selected.

*   Event Template: Hello World
*   Event Name: `hello`

Click on the **Create** button in the bottom right corner to proceed. You should now have a new test named `hello` next to the Test button. If so, click on the **Test** button to run the `hello` test.

```
Note: Why did we use the Hello World test? Because it doesn't matter! The function is not accepting any inputs, so the testing construct itself doesn't have any impact on the code. This will make more sense once we actually create the Roxie bot in the AWS Lex section.
```

The top area of the window will have a new area generated that contains the execution results of your test. If all goes well, this area will change to a light green color and state a result of succeeded. You can expand the details section to see the results in JSON format. In this example, had Roxie been asked to get cluster status by a user, she would have responded with "Your cluster is doing awesome."

![Test Content for get_cluster_status](/docs/images/get-cluster-status-test-content.jpg)

## Troubleshooting

Most of the issues discovered with AWS Lambda fall into two categories: networking or authorization. Check out the checklists below to get around any known stumbling points or submit an Issue on the Roxie repository on GitHub for more prescriptive assistance.

### Network Troubleshooting

Some items to investigate on the network:

*   Have you checked your security device logs to see if the function traffic is being received end-to-end?
*   Does your function have the correct network configuration values for the VPC, subnets, and security group?
*   Does your function correctly use the role that you attached the `lambda_vpc` policy against?
*   Have you checked to make sure that the VPC subnets can talk to your Rubrik cluster?
*   Are there any firewalls blocking the traffic?
*   Is your NAT Gateway configured properly?
*   Are you able to build an EC2 instance in the VPC network that your function is trying to use in order to validate that it can communicate with your Rubrik cluster?
*   Have you looked at the CloudWatch Logs and/or CloudTrail details to see where the function is failing?

### Authorization Troubleshooting

Some items to investigate for authorization:

*   Have you checked the Rubrik cluster to see why the authorization request failed?
*   Have you perhaps limited the Roxie account to the point where it can't get cluster status?
*   Have you tried using an Administrative token to see if the issue is with the scope or permissiveness of your account?

---

# AWS Lex

Amazon Lex is an AWS service for building conversational interfaces into applications using voice and text. With Amazon Lex, the same deep learning engine that powers Amazon Alexa is now available to any developer, enabling you to build sophisticated, natural language chatbots into your new and existing applications. Amazon Lex provides the deep functionality and flexibility of natural language understanding (NLU) and automatic speech recognition (ASR) to enable you to build highly engaging user experiences with lifelike, conversational interactions and create new categories of products.

Roxie uses Amazon Lex to come alive and answer your questions. For every question you ask, she will attempt to look through all of the available decision points to see if there is a Lambda function that can be called to answer your question. Consider Lex to be Roxie's brain - she accepts verbal inputs, uses Lambda to find the answer, and then responds as an output.

![Roxie Architecture](/docs/images/architecture.jpg)

## Bot Creation

Start by making a new custom bot for Roxie to use.

*   AWS Lex > Bots > Create > Custom Bot
    *   Bot Name: Roxie
    *   Language: English (US)
    *   Output Voice: Salli
    *   Session Timeout: 1 Minute
    *   IAM Role: `AWSServiceRoleForLexBots` (_Automatically created on your behalf_)
    *   COPPA: No

You now have a new bot to use for Roxie. That wasn't hard at all!

## Intent Creation

An intent represents an action that the user wants to perform. Each intent should map to a command that you wish to see Roxie respond to.

1.  Click on the **Create Intent** button to start the workflow.
1.  Next, select **Create Intent** in the list menu.
1.  Finally, give this intent a name. We're going to use the name `cluster_status`.

You now have your first intent. Let's dive deeper into the configuration areas of focus: sample utterances and fulfillment.

### Sample Utterances

A sample utterance is the words or phrases that Roxie will listen for when determining what to do. Since this guide is all about getting cluster status, it would make sense to pick 3-5 of the most common ways that question can be asked and use them as sample utterances.

We'll pick several phrases from the `get_cluster_status.py` example and enter them below:

1.  Status of my cluster
1.  Is the cluster doing okay
1.  How is my cluster doing
1.  What is the cluster status
1.  What is the status of the cluster

![Sample Utterances](/docs/images/sample-utterances.jpg)

If the spoken command matches one of the sample utterances, the intent will be chosen. Roxie will then look at the fulfillment to see what to do next.

### Fulfillment

The business logic required to fulfill the user's intent is called a fulfillment. Essentially, if Roxie hears the phrase "What is the status of the cluster", she then needs to know how to respond. We're going to use the Lambda function created earlier to programmatically reach out to Rubrik cluster's API and retrieve information needed to respond.

Navigate to the fulfillment section of the intent, choose the **AWS Lambda function** radio button, and then use the drop down menu to pick the `get_cluster_status` function with the version default of Latest.

```
Note: You will most likely see a prompt stating "You are about to give Amazon Lex permission to invoke your Lambda Function." Click on the OK button to continue.
```

![Fulfillment](/docs/images/fulfillment.jpg)

Configuration for this intent is now done. Make sure to scroll all the way to the bottom of the intent window and click **Save Intent**.

## Bot Build

With the configuration out of the way, it's now time to build the bot with the new intent. Locate and click the **Build **button on the top right area of the intent page. It may take several minutes before your bot's build is done.

```
Note: You will most likely see a warning that says "You can continue editing your bot while the build is in progress. You can start testing your bot after the build completes." You can safely accept this warning by clicking the new Build button that is presented to you.
```

Once the build finishes, the message shown in the image below will most likely appear.

![Roxie Build Successful](/docs/images/roxie-build-successful.jpg)

## Bot Testing

Let's test the bot and intent to see if it works properly. Locate the hidden sidebar on the right side of the intent window, which says "Test Chatbot", and expand it. This testing area allows you to either speak or type a sample utterance and see if the correct response is returned. This is the one time it's perfectly acceptable to talk to yourself out loud!

If you have a microphone handy, click on the small microphone button shown in the image below and speak an utterance. When done, click the small microphone button again to signal that you are done talking.

```
Note: If you don't have a microphone handy, you can type into the chat box instead.
```

![Test Bot Screen](/docs/images/test-bot-1.jpg)

Roxie will interpret the audio input and try to match it against an utterance contained in one or more intents. The response data contains a lot of great information to help understand what Lex is doing for Roxie. Click on the **Detail** radio button as shown in the image below to see more.

![Test Bot Screen](/docs/images/test-bot-2.jpg)

*   Intent-Name: This should match the intent based on the sample utterances provided int the intent. In this case, you should see the value of `cluster_status`.
*   Dialog-State: Fulfilled
*   Input-Transcript: The words you spoke into the microphone.
*   Message: The response from Roxie, as generated by the Lambda function `get_cluster_status`.

Feel free to experiment with different inputs to tweak your sample utterances and add any that you think are missing.