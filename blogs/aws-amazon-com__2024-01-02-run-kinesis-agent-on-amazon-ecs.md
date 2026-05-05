---
title: "Run Kinesis Agent on Amazon ECS"
url: "https://aws.amazon.com/blogs/big-data/run-kinesis-agent-on-amazon-ecs/"
date: "Tue, 02 Jan 2024 16:26:31 +0000"
author: "Buddhike de Silva"
feed_url: "https://aws.amazon.com/blogs/big-data/category/analytics/amazon-kinesis/kinesis-data-firehose/feed/"
---
<p><em><strong>February 9, 2024:</strong> Amazon Kinesis Data Firehose has been renamed to Amazon Data Firehose. Read the <a href="https://aws.amazon.com/about-aws/whats-new/2024/02/amazon-data-firehose-formerly-kinesis-data-firehose/" rel="noopener" target="_blank">AWS What’s New post</a> to learn more.</em></p> 
<p><a href="https://docs.aws.amazon.com/streams/latest/dev/writing-with-agents.html" rel="noopener" target="_blank">Kinesis Agent</a> is a standalone Java software application that offers a straightforward way to collect and send data to <a href="https://aws.amazon.com/kinesis/data-streams/" rel="noopener" target="_blank">Amazon Kinesis Data Streams</a> and <a href="https://aws.amazon.com/kinesis/data-firehose/" rel="noopener" target="_blank">Amazon Kinesis Data Firehose</a>. The agent continuously monitors a set of files and sends new data to the desired destination. The agent handles file rotation, checkpointing, and retry upon failures. It delivers all of your data in a reliable, timely, and simple manner. It also emits <a href="http://aws.amazon.com/cloudwatch" rel="noopener" target="_blank">Amazon CloudWatch</a> metrics to help you better monitor and troubleshoot the streaming process.</p> 
<p>This post describes the steps to send data from a containerized application to Kinesis Data Firehose using Kinesis Agent. More specifically, we show how to run Kinesis Agent as a sidecar container for an application running in <a href="https://aws.amazon.com/ecs/" rel="noopener" target="_blank">Amazon Elastic Container Service</a> (Amazon ECS). After the data is in Kinesis Data Firehose, it can be sent to any supported <a href="https://docs.aws.amazon.com/firehose/latest/dev/create-destination.html" rel="noopener" target="_blank">destination</a>, such as <a href="https://aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon Simple Storage Service</a> (Amazon S3).</p> 
<p>In order to present the key points required for this setup, we assume that you are familiar with Amazon ECS and working with containers. We also avoid the implementation details and packaging process of our test data generation application, referred to as the producer.</p> 
<h2>Solution overview</h2> 
<p>As depicted in the following figure, we configure a Kinesis Agent container as a sidecar that can read files created by the producer container. In this instance, the producer and Kinesis Agent containers share data via a <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/bind-mounts.html" rel="noopener" target="_blank">bind mount</a> in Amazon ECS.</p> 
<p><img alt="Solution design diagram" class="alignnone wp-image-58229 size-full" height="578" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/12/20/bdb-3919-arch-diag-1.png" width="1716" /></p> 
<h2>Prerequisites</h2> 
<p>You should satisfy the following prerequisites for the successful completion of this task:</p> 
<ul> 
 <li>Familiarity working with containers and Amazon ECS</li> 
 <li><a href="https://docs.docker.com/engine/install/" rel="noopener" target="_blank">Docker Desktop</a> installed</li> 
 <li>The <a href="http://aws.amazon.com/cli" rel="noopener" target="_blank">AWS Command Line Interface</a> (AWS CLI) <a href="https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html" rel="noopener" target="_blank">installed</a></li> 
 <li>An ECS cluster</li> 
 <li>An <a href="http://aws.amazon.com/ecr/" rel="noopener" target="_blank">Amazon Elastic Container Registry</a> (Amazon ECR) repository to store the Kinesis Agent container image</li> 
</ul> 
<p>With these prerequisites in place, you can begin next step to package a Kinesis Agent and your desired agent configuration as a container in your local development machine.</p> 
<h2>Create a Kinesis Agent configuration file</h2> 
<p>We use the <a href="https://docs.aws.amazon.com/firehose/latest/dev/writing-with-agents.html#agent-config-settings" rel="noopener" target="_blank">Kinesis Agent configuration file</a> to configure the source and destination, among other data transfer settings. The following code uses the minimal configuration required to read the contents of files matching <code>/var/log/producer/*.log</code> and publish them to a Kinesis Data Firehose delivery stream called <code>kinesis-agent-demo</code>:</p> 
<div class="hide-language"> 
 <pre><code class="lang-json">{
    "firehose.endpoint": "firehose.ap-southeast-2.amazonaws.com",
    "flows": [
        {
            "deliveryStream": "kinesis-agent-demo",
            "filePattern": "/var/log/producer/*.log"
        }
    ]
}</code></pre> 
</div> 
<h2>Create a container image for Kinesis Agent</h2> 
<p>To deploy Kinesis Agent as a sidecar in Amazon ECS, you first have to package it as a container image. The container must have Kinesis Agent, <code>which</code> and <code>find</code> binaries, and the Kinesis Agent configuration file that you prepared earlier. Its entry point must be configured using the <code>start-aws-kinesis-agent</code> script. This command is installed when you run the <code>yum install aws-kinesis-agent</code> step. The resulting Dockerfile should look as follows:</p> 
<div class="hide-language"> 
 <pre><code class="lang-yaml">FROM amazonlinux

RUN yum install -y aws-kinesis-agent which findutils
COPY agent.json /etc/aws-kinesis/agent.json

CMD ["start-aws-kinesis-agent"]</code></pre> 
</div> 
<p>Run the <code>docker build</code> command to build this container:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">docker build -t kinesis-agent .</code></pre> 
</div> 
<p>After the image is built, it should be pushed to a container registry like Amazon ECR so that you can reference it in the next section.</p> 
<h2>Create an ECS task definition with Kinesis Agent and the application container</h2> 
<p>Now that you have Kinesis Agent packaged as a container image, you can use it in your ECS task definitions to run as sidecar. To do that, you create an ECS task definition with your application container (called <code>producer</code>) and Kinesis Agent container. All containers in a task definition are scheduled on the same container host and therefore can share resources such as bind mounts.</p> 
<p>In the following sample container definition, we use a bind mount called <code>logs_dir</code> to share a directory between the <code>producer</code> container and <code>kinesis-agent</code> container.</p> 
<p>You can use the following template as a starting point, but be sure to change <code>taskRoleArn</code> and <code>executionRoleArn</code> to valid IAM roles in your AWS account. In this instance, the IAM role used for <code>taskRoleArn</code> must have write permissions to Kinesis Data Firehose that you specified earlier in the <code>agent.json</code> file. Additionally, make sure that the ECR image paths and <code>awslogs-region</code> are modified as per your AWS account.</p> 
<div class="hide-language"> 
 <pre><code class="lang-json">{
    "family": "kinesis-agent-demo",
    "taskRoleArn": "arn:aws:iam::111111111:role/kinesis-agent-demo-task-role",
    "executionRoleArn": "arn:aws:iam::111111111:role/kinesis-agent-test",
    "networkMode": "awsvpc",
    "containerDefinitions": [
        {
            "name": "producer",
            "image": "111111111.dkr.ecr.ap-southeast-2.amazonaws.com/producer:latest",
            "cpu": 1024,
            "memory": 2048,
            "essential": true,
            "command": [
                "-output",
                "/var/log/producer/test.log"
            ],
            "mountPoints": [
                {
                    "sourceVolume": "logs_dir",
                    "containerPath": "/var/log/producer",
                    "readOnly": false
                }
            ],
            "logConfiguration": {
                "logDriver": "awslogs",
                "options": {
                    "awslogs-create-group": "true",
                    "awslogs-group": "producer",
                    "awslogs-stream-prefix": "producer",
                    "awslogs-region": "ap-southeast-2"
                }
            }
        },
        {
            "name": "kinesis-agent",
            "image": "111111111.dkr.ecr.ap-southeast-2.amazonaws.com/kinesis-agent:latest",
            "cpu": 1024,
            "memory": 2048,
            "essential": true,
            "mountPoints": [
                {
                    "sourceVolume": "logs_dir",
                    "containerPath": "/var/log/producer",
                    "readOnly": true
                }
            ],
            "logConfiguration": {
                "logDriver": "awslogs",
                "options": {
                    "awslogs-create-group": "true",
                    "awslogs-group": "kinesis-agent",
                    "awslogs-stream-prefix": "kinesis-agent",
                    "awslogs-region": "ap-southeast-2"
                }
            }
        }
    ],
    "volumes": [
        {
            "name": "logs_dir"
        }
    ],
    "requiresCompatibilities": [
        "FARGATE"
    ],
    "cpu": "2048",
    "memory": "4096"
}</code></pre> 
</div> 
<p>Register the task definition with the following command:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">aws ecs register-task-definition --cli-input-json file://./task-definition.json</code></pre> 
</div> 
<h2>Run a new ECS task</h2> 
<p>Finally, you can run a new ECS task using the task definition you just created using the <code>aws ecs run-task</code> command. When the task is started, you should be able to see two containers running under that task on the Amazon ECS console.</p> 
<p><img alt="Amazon ECS console screenshot" class="alignnone wp-image-58227 size-full" height="1142" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/12/20/bdb-3919-ecs-screenshot.png" width="1982" /></p> 
<h2>Conclusion</h2> 
<p>This post showed how straightforward it is to run Kinesis Agent in a containerized environment. Although we used Amazon ECS as our container orchestration service in this post, you can use a Kinesis Agent container in other environments such as <a href="https://aws.amazon.com/eks/" rel="noopener" target="_blank">Amazon Elastic Kubernetes Service</a> (Amazon EKS).</p> 
<p>To learn more about using Kinesis Agent, refer to <a href="https://docs.aws.amazon.com/streams/latest/dev/writing-with-agents.html" rel="noopener" target="_blank">Writing to Amazon Kinesis Data Streams Using Kinesis Agent</a>. For more information about Amazon ECS, refer to the <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html" rel="noopener" target="_blank">Amazon ECS Developer Guide</a>.</p> 
<hr /> 
<h3>About the Author</h3> 
<p><img alt="" class="alignleft wp-image-58251 size-full" height="104" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/12/21/author_buddhiks-100.jpg" width="100" /><strong>Buddhike de Silva&nbsp;</strong>is a Senior Specialist Solutions Architect at Amazon Web Services. Buddhike helps customers run large scale streaming analytics workloads on AWS and make the best out of their cloud journey.</p>
