# Getting Started

Open up a web browser and go to [http://bit.ly/tgas-syd](http://bit.ly/tgas-syd). Login using your group number and the password `Rubrik123!`

For example, if you are in group 1 then your username is group1 and the password is `Rubrik123!`

![login screen](/docs/images/login.jpg)

# AWS Lambda

AWS Lambda is a compute service that lets you run code without provisioning or managing servers. AWS Lambda executes your code only when needed and scales automatically, from a few requests per day to thousands per second. All you need to do is supply your code in one of the languages that AWS Lambda supports (currently Node.js, Java, C#, Go and Python).

For this exercise, we're going to give our chatbot the power to get status on a software cluster. This is driven by a Lambda function written for the Python 2.7 runtime and is available in the repository on GitHub.

## Function Creation

Start by navigating to the AWS Lambda service and clicking the **Create Function** button and choosing the **Author From Scratch** option.

*   Name: `get_sla_compliance`
*   Runtime: Python 2.7
*   Role: Create a new role from one or more templates
*   Role Name: `roxie_lambda`
*   Policy Templates: Leave blank

Click on the **Create Function** button at the bottom right corner to proceed. The creation process may take a little while to complete.

Once finished, you will be taken to the `get_sla_compliance` function page. There's a lot of menu items on this page - don't be alarmed! We're going to come back to this page in a moment.

## Function Configuration

There are only three sections to edit in the Lambda function page:

1.  [Function Code](#function-code): This is where the Python code for getting cluster status is loaded.
1.  [Environmental Variables](#environmental-variables): This is where the connection information for the backend Rubrik cluster is stored.
1.  [Network](#network): This is where the VPC network is selected.

### Function Code

For this guide, we're going to use the contents of the `get_sla_compliance.py` file found in the repository on GitHub. Open the [`get_sla_compliance.py` file](/intents/get_sla_compliance.py) and copy the code.

Go back to AWS and locate the Function Code window and delete any code that already exists in the lambda_function window. Then, paste the contents of the `get_sla_compliance.py` file into the Function Code window. The resulting output should look like the image below:

![Function Code for get_sla_compliance](/docs/images/get-cluster-status-function-code.jpg)

You can now click the **Save** button in the upper right corner of the window to save your progress. It should change from orange to a greyed-out color upon success.

### Environmental Variables

Next, the function needs to know how to connect to your cluster. We feel it's better to keep static information out of the function code.

For this step, it's important to add three environment variables: `CLUSTER_IP`, `USERNAME`, and `PASSWORD`. This can be done by scrolling down below the Function Code area and locating the Environmental Variables area.

1.  `CLUSTER_IP`: 54.215.205.88
1.  `PASSWORD`: Rubrik123!!
1.  `USERNAME`: admin

![Environmental Variables for get_sla_compliance](/docs/images/environmental-variables.jpg)

With those values updated, you can now click the **Save** button in the upper right corner of the window to save your progress. It should change from orange to a greyed-out color upon success.

## Function Testing

Before proceeding further, it's a good idea to test your Lambda function to make sure that the code and networking configuration are functioning properly. To do this, use the built in Test functionality in the function.

Click the **Test** button in the top right corner. Next, make sure the **Create New Test Event** radio button is selected.

*   Event Template: Hello World
*   Event Name: `hello`

Click on the **Create** button in the bottom right corner to proceed. You should now have a new test named `hello` next to the Test button. If so, click on the **Test** button to run the `hello` test.

```
Note: Why did we use the Hello World test? Because it doesn't matter! The function is not accepting any inputs, so the testing construct itself doesn't have any impact on the code. This will make more sense once we actually create the bot in the AWS Lex section.
```

The top area of the window will have a new area generated that contains the execution results of your test. If all goes well, this area will change to a light green color and state a result of succeeded. You can expand the details section to see the results in JSON format. In this example, had Roxie been asked to get cluster status by a user, she would have responded with "Your cluster is doing awesome."

![Test Content for get_sla_compliance](/docs/images/get-cluster-status-test-content.jpg)

# AWS Lex

Amazon Lex is an AWS service for building conversational interfaces into applications using voice and text. With Amazon Lex, the same deep learning engine that powers Amazon Alexa is now available to any developer, enabling you to build sophisticated, natural language chatbots into your new and existing applications. Amazon Lex provides the deep functionality and flexibility of natural language understanding (NLU) and automatic speech recognition (ASR) to enable you to build highly engaging user experiences with lifelike, conversational interactions and create new categories of products.

The chatbot uses Amazon Lex to come alive and answer your questions. For every question you ask, it will attempt to look through all of the available decision points to see if there is a Lambda function that can be called to answer your question. Consider Lex to be the chatbot's brain - it accepts verbal inputs, uses Lambda to find the answer, and then responds as an output.

![Roxie Architecture](/docs/images/architecture.jpg)

## Bot Selection

At the top of AWS, click **Services** and search for Lex. Make sure you are in the **N. Virginia** region. A bot called `Superhero_Roxie` has been created for this exercise.

![Select the bot](/docs/images/bot-select.jpg)

Click `Superhero_Roxie`.

## Intent Creation

An intent represents an action that the user wants to perform. Each intent should map to a command that you wish to see Roxie respond to.

![Create intent](/docs/images/create-intent.jpg)

1.  Click on the **Create Intent** button to start the workflow.
1.  Next, select **Create Intent** in the list menu.
1.  Finally, give this intent a name. We're going to use the name `get_sla_compliance`.

You now have your first intent. Let's dive deeper into the configuration areas of focus: sample utterances and fulfillment.

### Sample Utterances

A sample utterance is the words or phrases that Roxie will listen for when determining what to do. Since this guide is all about getting cluster status, it would make sense to pick 3-5 of the most common ways that question can be asked and use them as sample utterances.

We'll pick several phrases from the `get_sla_compliance.py` example and enter them below:

1.  How are my SLAs doing
1.  Are my SLAs in compliance
1.  Read out SLA compliance summary
1.  Give me SLA compliance summary
1.  Get me SLA compliance summary
1.  How many SLAs are in compliance

![Sample Utterances](/docs/images/sample-utterances.jpg)

If the spoken command matches one of the sample utterances, the intent will be chosen. The bot will then look at the fulfillment to see what to do next.

### Fulfillment

The business logic required to fulfill the user's intent is called a fulfillment. Essentially, if the chatbot hears the phrase "What is the status of the cluster", it then needs to know how to respond. We're going to use the Lambda function created earlier to programmatically reach out to Rubrik cluster's API and retrieve information needed to respond.

Navigate to the fulfillment section of the intent, choose the **AWS Lambda function** radio button, and then use the drop down menu to pick the `get_sla_compliance` function with the version default of Latest.

```
Note: You will most likely see a prompt stating "You are about to give Amazon Lex permission to invoke your Lambda Function." Click on the OK button to continue.
```

![Fulfillment](/docs/images/fulfillment.jpg)

Configuration for this intent is now done. Make sure to scroll all the way to the bottom of the intent window and click **Save Intent**.

## Bot Build

With the configuration out of the way, it's now time to build the bot with the new intent. Locate and click the **Build** button on the top right area of the intent page. It may take several minutes before your bot's build is done.

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

*   Intent-Name: This should match the intent based on the sample utterances provided int the intent. In this case, you should see the value of `get_sla_compliance`.
*   Dialog-State: Fulfilled
*   Input-Transcript: The words you spoke into the microphone.
*   Message: The response from Roxie, as generated by the Lambda function `get_sla_compliance`.

Feel free to experiment with different inputs to tweak your sample utterances and add any that you think are missing.