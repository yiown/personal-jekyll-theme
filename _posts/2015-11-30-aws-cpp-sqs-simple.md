---
layout: post
section-type: post
title: AWS CPP SQS Simple Client
category: AWS
tags: [ 'aws', 'cpp', 'sqs', 'c++14' ]
---
AWS is such a wonderful platform, and code agnostic, but it is surprisingly not supported from C++ consumers.
Up until now, since recently Amazon released a complete set of AWS interfaces for C++: [AWS SDK CPP](https://github.com/awslabs/aws-sdk-cpp).

I had to give it a try and wrote the most simple test for SQS.\\
Remember to have your AWS keys in context variables:

~~~ Bash
export AWS_ACCESS_KEY=<your access key>
export AWS_SECRET_ACCESS_KEY=<your secret key>
~~~
 \\
To run the test, pass it the SQS queue URL by parameter.\\
Here is the complete code:

~~~ Cpp
#include <iostream>
#include <aws/sqs/SQSClient.h>
#include <aws/core/auth/AWSCredentialsProviderChain.h>
#include <aws/sqs/model/SendMessageRequest.h>
#include <aws/sqs/model/ReceiveMessageRequest.h>
#include <aws/sqs/model/DeleteMessageRequest.h>
#include <jsoncpp/json/value.h>
#include <jsoncpp/json/writer.h>
#include <jsoncpp/json/reader.h>

/**
 * @brief Main entry point.
 * @param argc Argument count.
 * @param argv Arguments.
 * @return Status.
 */
int main(int argc, char* argv[]) {
    // check arguments
    if(argc < 1) {
        std::cout << "Missing parameter queueUrl." << std::endl;
        std::cout << "Example: sqsTest \"https://sqs.us-east-1.amazonaws.com/123456/myQueue\"." << std::endl;
        return 0;
    }
    // extract data from arguments
    std::string queueUrl{argv[1]};

    // test json object
    Json::Value root;
    root["meaningOfLife"] = 42;
    root["recommendation"] = "take a towel";
    std::string json = Json::FastWriter().write(root);

    // create sqs client
    Aws::Client::ClientConfiguration config;
    config.scheme = Aws::Http::Scheme::HTTPS;
    config.region = Aws::Region::US_EAST_1;
    Aws::SQS::SQSClient sqsClient{config};

    try {
        // send message
        Aws::SQS::Model::SendMessageRequest sendMessageRequest;
        sendMessageRequest.SetQueueUrl(queueUrl);
        sendMessageRequest.SetMessageBody(json);
        Aws::SQS::Model::SendMessageOutcome sendMessageOutcome = sqsClient.SendMessage(sendMessageRequest);
        if(!sendMessageOutcome.IsSuccess() || sendMessageOutcome.GetResult().GetMessageId().length() == 0) {
            std::cout << "Error on send: " << sendMessageOutcome.GetError().GetMessage() << std::endl;
            return 1;
        }

        // receive message
        Aws::SQS::Model::ReceiveMessageRequest receiveMessageRequest;
        receiveMessageRequest.SetQueueUrl(queueUrl);
        receiveMessageRequest.SetMaxNumberOfMessages(1);
        receiveMessageRequest.AddMessageAttributeNames("All");
        Aws::SQS::Model::ReceiveMessageOutcome receiveMessageOutcome = sqsClient.ReceiveMessage(receiveMessageRequest);
        if(!receiveMessageOutcome.IsSuccess() || receiveMessageOutcome.GetResult().GetMessages().size() == 0) {
            std::cout << "Error on receive: " << receiveMessageOutcome.GetError().GetMessage() << std::endl;
            return 2;
        }
        Aws::SQS::Model::Message msg = receiveMessageOutcome.GetResult().GetMessages()[0];

        // delete message
        Aws::SQS::Model::DeleteMessageRequest deleteMessageRequest;
        deleteMessageRequest.SetQueueUrl(queueUrl);
        deleteMessageRequest.SetReceiptHandle(msg.GetReceiptHandle());
        Aws::SQS::Model::DeleteMessageOutcome deleteMessageOutcome = sqsClient.DeleteMessage(deleteMessageRequest);
        if(!deleteMessageOutcome.IsSuccess()) {
            std::cout << "Error on delete: " << deleteMessageOutcome.GetError().GetMessage() << std::endl;
            return 3;
        }

        // check message
        Json::Value received;
        if(!Json::Reader().parse(msg.GetBody(), received, false)
                || received["meaningOfLife"].asInt() != 42
                || received["recommendation"].asString() != "take a towel") {
            std::cout << "Received message not valid." << std::endl;
            return 4;
        }
    } catch(...) {
        std::cout << "Exception occurred." << std::endl;
    }

    return 0;
}
~~~

**glhf**
