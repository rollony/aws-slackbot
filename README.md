# aws-slackbot
Slackbot using AWS Lamdba and DynamoDB

1. Building the Dockerfile
2. Push to Amazon Elastic Container Registry
3. Creating an IAM Role for Lamdba function with suitable access permissions
4. Integration with Lambda function and API Gateway
5. Integration with DynamoDB for message logging

## Building the Dockerfile / Push to Amazon Elastic Container Registry

Configure access to AWS credentials using the CLI 
```shell
aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <account>.dkr.ecr.<region>.amazonaws.com
```

Build the current directory into a Docker image
```shell
docker build -t slackbot-test .
```

Add a suitable tag for the created image
```shell
docker tag slackbot-test:latest <account>.dkr.ecr.<region>.amazonaws.com/slackbot-test:latest
```

Once the image is ready, push to Elastic Container Registry
```shell
docker push <account>.dkr.ecr.<region>.amazonaws.com/slackbot-test:latest
```

## Creating an IAM Role for Lamdba function with suitable access permissions

The following policy serves the purpose to grant access of generating logs and allowing actions on the DynamoDB table that is to be created. </br>
Apply the policy to the role that will serve as the default execution role of the Lambda function that will be used.
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "logs:CreateLogGroup",
            "Resource": "arn:aws:logs:ap-northeast-2:<account>:*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "arn:aws:logs:ap-northeast-2:<account>:log-group:/aws/lambda/slackbot-test:*"
        },
        {
            "Effect": "Allow",
            "Action": "dynamodb:*",
            "Resource": "arn:aws:dynamodb:ap-northeast-2:<account>:table/todos"
        }
    ]
}
```

## Integration with Lambda function and API Gateway

Create a Lambda function using the container image which we previously pushed into our ECR. </br>
Edit the default execution role to use the existing one made earlier, so the Lambda function is given the necessary permissions. </br>
Create an API Gateway (HTTP API is recommended) that integrates the Lambda function we just created and edit the configurated routes to only allow POST methods. </br>
After configuring the API Gateway, the function overview in the Lambda section will look like this. </br>
![slackbot-jpg3](https://user-images.githubusercontent.com/30142314/170823750-d8cf841d-cd72-4d1e-84eb-92ec33170144.JPG)



## Integration with DynamoDB for message logging

Create a DynamoDB table with the exact name stated in the policy we created earlier. </br>
It will now be possible to write messages into the DynamoDB table by using the mention function in Slack.

![slackbot-jpg](https://user-images.githubusercontent.com/30142314/170823645-b72663bb-ef2e-43f9-8932-49f1a0a4fa52.JPG)

Messages will appear in the DynamoDB table as the below image indicates.
![slackbot-jpg2](https://user-images.githubusercontent.com/30142314/170823714-ed4c79e7-7310-464b-8d3b-c645d872e386.JPG)

## References
https://github.com/antonputra/tutorials/tree/main/lessons/076
