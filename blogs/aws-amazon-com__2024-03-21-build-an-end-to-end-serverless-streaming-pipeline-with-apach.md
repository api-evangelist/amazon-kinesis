---
title: "Build an end-to-end serverless streaming pipeline with Apache Kafka on Amazon MSK using Python"
url: "https://aws.amazon.com/blogs/big-data/build-an-end-to-end-serverless-streaming-pipeline-with-apache-kafka-on-amazon-msk-using-python/"
date: "Thu, 21 Mar 2024 15:03:17 +0000"
author: "Masudur Rahaman Sayem"
feed_url: "https://aws.amazon.com/blogs/big-data/category/analytics/amazon-kinesis/kinesis-data-firehose/feed/"
---
<p>The volume of data generated globally continues to surge, from gaming, retail, and finance, to manufacturing, healthcare, and travel. Organizations are looking for more ways to quickly use the constant inflow of data to innovate for their businesses and customers. They have to reliably capture, process, analyze, and load the data into a myriad of data stores, all in real time.</p> 
<p>Apache Kafka is a popular choice for these real-time streaming needs. However, it can be challenging to set up a Kafka cluster along with other data processing components that scale automatically depending on your application’s needs. You risk under-provisioning for peak traffic, which can lead to downtime, or over-provisioning for base load, leading to wastage. AWS offers multiple serverless services like <a href="https://aws.amazon.com/msk/" rel="noopener" target="_blank">Amazon Managed Streaming for Apache Kafka</a> (Amazon MSK), <a href="https://aws.amazon.com/firehose/" rel="noopener" target="_blank">Amazon Data Firehose</a>, <a href="https://aws.amazon.com/dynamodb/" rel="noopener" target="_blank">Amazon DynamoDB</a>, and <a href="https://aws.amazon.com/pm/lambda/" rel="noopener" target="_blank">AWS Lambda</a> that scale automatically depending on your needs.</p> 
<p>In this post, we explain how you can use some of these services, including <a href="https://docs.aws.amazon.com/msk/latest/developerguide/serverless.html" rel="noopener" target="_blank">MSK Serverless</a>, to build a serverless data platform to meet your real-time needs.</p> 
<h2>Solution overview</h2> 
<p>Let’s imagine a scenario. You’re responsible for managing thousands of modems for an internet service provider deployed across multiple geographies. You want to monitor the modem connectivity quality that has a significant impact on customer productivity and satisfaction. Your deployment includes different modems that need to be monitored and maintained to ensure minimal downtime. Each device transmits thousands of 1 KB records every second, such as CPU usage, memory usage, alarm, and connection status. You want real-time access to this data so you can monitor performance in real time, and detect and mitigate issues quickly. You also need longer-term access to this data for machine learning (ML) models to run predictive maintenance assessments, find optimization opportunities, and forecast demand.</p> 
<p>Your clients that gather the data onsite are written in Python, and they can send all the data as Apache Kafka topics to Amazon MSK. For your application’s low-latency and real-time data access, you can use <a href="https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Programming.html" rel="noopener" target="_blank">Lambda and DynamoDB</a>. For longer-term data storage, you can use managed serverless connector service <a href="https://docs.aws.amazon.com/firehose/latest/dev/create-destination.html" rel="noopener" target="_blank">Amazon Data Firehose</a> to send data to your data lake.</p> 
<p>The following diagram shows how you can build this end-to-end serverless application.</p> 
<p><img alt="end-to-end serverless application" class="alignnone size-full wp-image-47247" height="378" src="https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2024/03/19/bdb3981-1.png" width="887" /></p> 
<p>Let’s follow the steps in the following sections to implement this architecture.</p> 
<h2>Create a serverless Kafka cluster on Amazon MSK</h2> 
<p>We use Amazon MSK to ingest real-time telemetry data from modems. Creating a serverless Kafka cluster is straightforward on Amazon MSK. It only takes a few minutes using the <a href="http://aws.amazon.com/console" rel="noopener" target="_blank">AWS Management Console</a> or AWS SDK. To use the console, refer to <a href="https://docs.aws.amazon.com/msk/latest/developerguide/serverless-getting-started.html" rel="noopener" target="_blank">Getting started using MSK Serverless clusters</a>. You create a serverless cluster, <a href="https://aws.amazon.com/iam/" rel="noopener" target="_blank">AWS Identity and Access Management</a> (IAM) role, and client machine.</p> 
<h2>Create a Kafka topic using Python</h2> 
<p>When your cluster and client machine are ready, SSH to your client machine and install Kafka Python and the MSK IAM library for Python.</p> 
<ul> 
 <li>Run the following commands to install Kafka Python and the <a href="https://docs.aws.amazon.com/msk/latest/developerguide/iam-access-control.html" rel="noopener" target="_blank">MSK IAM library</a>:</li> 
</ul> 
<div class="hide-language"> 
 <pre><code class="lang-python">pip install kafka-python

pip install aws-msk-iam-sasl-signer-python</code></pre> 
</div> 
<ul> 
 <li>Create a new file called <code>createTopic.py</code>.</li> 
 <li>Copy the following code into this file, replacing the <code>bootstrap_servers</code> and <code>region</code> information with the details for your cluster. For instructions on retrieving the <code>bootstrap_servers</code> information for your MSK cluster, see <a href="https://docs.aws.amazon.com/msk/latest/developerguide/msk-get-bootstrap-brokers.html" rel="noopener" target="_blank">Getting the bootstrap brokers for an Amazon MSK cluster</a>.</li> 
</ul> 
<div class="hide-language"> 
 <pre><code class="lang-python">from kafka.admin import KafkaAdminClient, NewTopic
from aws_msk_iam_sasl_signer import MSKAuthTokenProvider

# AWS region where MSK cluster is located
region= '&lt;UPDATE_AWS_REGION_NAME_HERE&gt;'

# Class to provide MSK authentication token
class MSKTokenProvider():
    def token(self):
        token, _ = MSKAuthTokenProvider.generate_auth_token(region)
        return token

# Create an instance of MSKTokenProvider class
tp = MSKTokenProvider()

# Initialize KafkaAdminClient with required configurations
admin_client = KafkaAdminClient(
    bootstrap_servers='&lt;UPDATE_BOOTSTRAP_SERVER_STRING_HERE&gt;',
    security_protocol='SASL_SSL',
    sasl_mechanism='OAUTHBEARER',
    sasl_oauth_token_provider=tp,
    client_id='client1',
)

# create topic
topic_name="mytopic"
topic_list =[NewTopic(name=topic_name, num_partitions=1, replication_factor=2)]
existing_topics = admin_client.list_topics()
if(topic_name not in existing_topics):
    admin_client.create_topics(topic_list)
    print("Topic has been created")
else:
    print("topic already exists!. List of topics are:" + str(existing_topics))
</code></pre> 
</div> 
<ul> 
 <li>Run the <code>createTopic.py</code> script to create a new Kafka topic called <code>mytopic</code> on your serverless cluster:</li> 
</ul> 
<div class="hide-language"> 
 <pre><code class="lang-python">python createTopic.py</code></pre> 
</div> 
<h2>Produce records using Python</h2> 
<p>Let’s generate some sample modem telemetry data.</p> 
<ul> 
 <li>Create a new file called <code>kafkaDataGen.py</code>.</li> 
 <li>Copy the following code into this file, updating the <code>BROKERS</code> and <code>region</code> information with the details for your cluster:</li> 
</ul> 
<div class="hide-language"> 
 <pre><code class="lang-python">from kafka import KafkaProducer
from aws_msk_iam_sasl_signer import MSKAuthTokenProvider
import json
import random
from datetime import datetime
topicname='mytopic'

BROKERS = '&lt;UPDATE_BOOTSTRAP_SERVER_STRING_HERE&gt;'
region= '&lt;UPDATE_AWS_REGION_NAME_HERE&gt;'
class MSKTokenProvider():
    def token(self):
        token, _ = MSKAuthTokenProvider.generate_auth_token(region)
        return token

tp = MSKTokenProvider()

producer = KafkaProducer(
    bootstrap_servers=BROKERS,
    value_serializer=lambda v: json.dumps(v).encode('utf-8'),
    retry_backoff_ms=500,
    request_timeout_ms=20000,
    security_protocol='SASL_SSL',
    sasl_mechanism='OAUTHBEARER',
    sasl_oauth_token_provider=tp,)

# Method to get a random model name
def getModel():
    products=["Ultra WiFi Modem", "Ultra WiFi Booster", "EVG2000", "Sagemcom 5366 TN", "ASUS AX5400"]
    randomnum = random.randint(0, 4)
    return (products[randomnum])

# Method to get a random interface status
def getInterfaceStatus():
    status=["connected", "connected", "connected", "connected", "connected", "connected", "connected", "connected", "connected", "connected", "connected", "connected", "down", "down"]
    randomnum = random.randint(0, 13)
    return (status[randomnum])

# Method to get a random CPU usage
def getCPU():
    i = random.randint(50, 100)
    return (str(i))

# Method to get a random memory usage
def getMemory():
    i = random.randint(1000, 1500)
    return (str(i))
    
# Method to generate sample data
def generateData():
    
    model=getModel()
    deviceid='dvc' + str(random.randint(1000, 10000))
    interface='eth4.1'
    interfacestatus=getInterfaceStatus()
    cpuusage=getCPU()
    memoryusage=getMemory()
    now = datetime.now()
    event_time = now.strftime("%Y-%m-%d %H:%M:%S")
    
    modem_data={}
    modem_data["model"]=model
    modem_data["deviceid"]=deviceid
    modem_data["interface"]=interface
    modem_data["interfacestatus"]=interfacestatus
    modem_data["cpuusage"]=cpuusage
    modem_data["memoryusage"]=memoryusage
    modem_data["event_time"]=event_time
    return modem_data

# Continuously generate and send data
while True:
    data =generateData()
    print(data)
    try:
        future = producer.send(topicname, value=data)
        producer.flush()
        record_metadata = future.get(timeout=10)
        
    except Exception as e:
        print(e.with_traceback())
</code></pre> 
</div> 
<ul> 
 <li>Run the <code>kafkaDataGen.py</code> to continuously generate random data and publish it to the specified Kafka topic:</li> 
</ul> 
<div class="hide-language"> 
 <pre><code class="lang-python">python&nbsp;kafkaDataGen.py</code></pre> 
</div> 
<h2>Store events in Amazon S3</h2> 
<p>Now you store all the raw event data in an <a href="http://aws.amazon.com/s3" rel="noopener" target="_blank">Amazon Simple Storage Service</a> (Amazon S3) data lake for analytics. You can use the same data to train ML models. The <a href="https://docs.aws.amazon.com/firehose/latest/dev/writing-with-msk.html" rel="noopener" target="_blank">integration with Amazon Data Firehose</a> allows Amazon MSK to seamlessly load data from your Apache Kafka clusters into an S3 data lake. Complete the following steps to continuously stream data from Kafka to Amazon S3, eliminating the need to build or manage your own connector applications:</p> 
<ul> 
 <li>On the Amazon S3 console, create a new bucket. You can also use an existing bucket.</li> 
 <li>Create a new folder in your S3 bucket called <code>streamingDataLake</code>.</li> 
 <li>On the Amazon MSK console, choose your MSK Serverless cluster.</li> 
 <li>On the <strong>Actions </strong>menu, choose <strong>Edit cluster policy</strong>.</li> 
</ul> 
<p><img alt="cluster policy" class="alignnone size-full wp-image-47258" height="346" src="https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2024/03/19/bdb3981-2.png" style="margin: 10px 0px 10px 0px;" width="1532" /></p> 
<ul> 
 <li>Select <strong>Include Firehose service principal</strong> and choose <strong>Save changes</strong>.</li> 
</ul> 
<p><img alt="firehose service principal" class="alignnone size-full wp-image-47268" height="792" src="https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2024/03/19/bdb3981-3.png" style="margin: 10px 0px 10px 0px;" width="819" /></p> 
<ul> 
 <li>On the <strong>S3 delivery</strong> tab, choose <strong>Create delivery stream</strong>.</li> 
</ul> 
<p><img alt="delivery stream" class="alignnone size-full wp-image-47269" height="328" src="https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2024/03/19/bdb3981-4.png" style="margin: 10px 0px 10px 0px;" width="1546" /></p> 
<ul> 
 <li>For <strong>Source</strong>, choose <strong>Amazon MSK</strong>.</li> 
 <li>For <strong>Destination</strong>, choose <strong>Amazon S3</strong>.</li> 
</ul> 
<p><img alt="source and destination" class="alignnone size-full wp-image-47270" height="373" src="https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2024/03/19/bdb3981-5.png" style="margin: 10px 0px 10px 0px;" width="817" /></p> 
<ul> 
 <li>For <strong>Amazon MSK cluster connectivity</strong>, select <strong>Private bootstrap brokers</strong>.</li> 
 <li>For <strong>Topic</strong>, enter a topic name (for this post, <code>mytopic</code>).</li> 
</ul> 
<p><img alt="source settings" class="alignnone size-full wp-image-47271" height="455" src="https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2024/03/19/bdb3981-6.png" style="margin: 10px 0px 10px 0px;" width="817" /></p> 
<ul> 
 <li>For <strong>S3 bucket</strong>, choose <strong>Browse</strong> and choose your S3 bucket.</li> 
 <li>Enter <code>streamingDataLake</code> as your S3 bucket prefix.</li> 
 <li>Enter <code>streamingDataLakeErr</code> as your S3 bucket error output prefix.</li> 
</ul> 
<p><img alt="destination settings" class="alignnone size-full wp-image-47272" height="640" src="https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2024/03/19/bdb3981-7.png" style="margin: 10px 0px 10px 0px;" width="819" /></p> 
<ul> 
 <li>Choose <strong>Create delivery stream</strong>.</li> 
</ul> 
<p><img alt="create delivery stream" class="alignnone size-full wp-image-47273" height="171" src="https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2024/03/19/bdb3981-8.png" style="margin: 10px 0px 10px 0px;" width="831" /></p> 
<p>You can verify that the data was written to your S3 bucket. You should see that the <code>streamingDataLake</code> directory was created and the files are stored in partitions.</p> 
<p><img alt="amazon s3" class="alignnone size-full wp-image-47274" height="583" src="https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2024/03/19/bdb3981-9.png" style="margin: 10px 0px 10px 0px;" width="616" /></p> 
<h2>Store events in DynamoDB</h2> 
<p>For the last step, you store the most recent modem data in DynamoDB. This allows the client application to access the modem status and interact with the modem remotely from anywhere, with low latency and high availability. <a href="https://docs.aws.amazon.com/lambda/latest/dg/with-msk.html" rel="noopener" target="_blank">Lambda seamlessly works with Amazon MSK</a>. Lambda internally polls for new messages from the event source and then synchronously invokes the target Lambda function. Lambda reads the messages in batches and provides these to your function as an event payload.</p> 
<p>Lets first create a table in DynamoDB. Refer to <a href="https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/api-permissions-reference.html" rel="noopener" target="_blank">DynamoDB API permissions: Actions, resources, and conditions reference</a> to verify that your client machine has the necessary permissions.</p> 
<ul> 
 <li>Create a new file called <code>createTable.py</code>.</li> 
 <li>Copy the following code into the file, updating the <code>region</code> information:</li> 
</ul> 
<div class="hide-language"> 
 <pre><code class="lang-python">import boto3
region='&lt;UPDATE_AWS_REGION_NAME_HERE&gt;'
dynamodb = boto3.client('dynamodb', region_name=region)
table_name = 'device_status'
key_schema = [
    {
        'AttributeName': 'deviceid',
        'KeyType': 'HASH'
    }
]
attribute_definitions = [
    {
        'AttributeName': 'deviceid',
        'AttributeType': 'S'
    }
]
# Create the table with on-demand capacity mode
dynamodb.create_table(
    TableName=table_name,
    KeySchema=key_schema,
    AttributeDefinitions=attribute_definitions,
    BillingMode='PAY_PER_REQUEST'
)
print(f"Table '{table_name}' created with on-demand capacity mode.")</code></pre> 
</div> 
<ul> 
 <li>Run the <code>createTable.py</code> script to create a table called <code>device_status</code> in DynamoDB:</li> 
</ul> 
<div class="hide-language"> 
 <pre><code class="lang-python">python createTable.py</code></pre> 
</div> 
<p>Now let’s configure the Lambda function.</p> 
<ul> 
 <li>On the Lambda console, choose <strong>Functions</strong> in the navigation pane.</li> 
 <li>Choose <strong>Create function</strong>.</li> 
 <li>Select <strong>Author from scratch</strong>.</li> 
 <li>For <strong>Function name</strong>¸ enter a name (for example, <code>my-notification-kafka</code>).</li> 
 <li>For <strong>Runtime</strong>, choose <strong>Python 3.11</strong>.</li> 
 <li>For <strong>Permissions</strong>, select <strong>Use an existing role</strong> and choose a role with <a href="https://docs.aws.amazon.com/lambda/latest/dg/with-msk.html#msk-permissions-iam-policy" rel="noopener" target="_blank">permissions to read from your cluster</a>.</li> 
 <li>Create the function.</li> 
</ul> 
<p>On the Lambda function configuration page, you can now configure sources, destinations, and your application code.</p> 
<ul> 
 <li>Choose <strong>Add trigger</strong>.</li> 
 <li>For <strong>Trigger configuration</strong>, enter <code>MSK</code> to configure Amazon MSK as a trigger for the Lambda source function.</li> 
 <li>For <strong>MSK cluster</strong>, enter <code>myCluster</code>.</li> 
 <li>Deselect <strong>Activate trigger</strong>, because you haven’t configured your Lambda function yet.</li> 
 <li>For <strong>Batch size</strong>, enter <code>100</code>.</li> 
 <li>For <strong>Starting position</strong>, choose <strong>Latest</strong>.</li> 
 <li>For <strong>Topic name</strong>¸ enter a name (for example, <code>mytopic</code>).</li> 
 <li>Choose <strong>Add</strong>.</li> 
 <li>On the Lambda function details page, on the <strong>Code </strong>tab, enter the following code:</li> 
</ul> 
<div class="hide-language"> 
 <pre><code class="lang-python">import base64
import boto3
import json
import os
import random

def convertjson(payload):
    try:
        aa=json.loads(payload)
        return aa
    except:
        return 'err'

def lambda_handler(event, context):
    base64records = event['records']['mytopic-0']
    
    raw_records = [base64.b64decode(x["value"]).decode('utf-8') for x in base64records]
    
    for record in raw_records:
        item = json.loads(record)
        deviceid=item['deviceid']
        interface=item['interface']
        interfacestatus=item['interfacestatus']
        cpuusage=item['cpuusage']
        memoryusage=item['memoryusage']
        event_time=item['event_time']
        
        dynamodb = boto3.client('dynamodb')
        table_name = 'device_status'
        item = {
            'deviceid': {'S': deviceid},  
            'interface': {'S': interface},               
            'interface': {'S': interface},
            'interfacestatus': {'S': interfacestatus},
            'cpuusage': {'S': cpuusage},          
            'memoryusage': {'S': memoryusage},
            'event_time': {'S': event_time},
        }
        
        # Write the item to the DynamoDB table
        response = dynamodb.put_item(
            TableName=table_name,
            Item=item
        )
        
        print(f"Item written to DynamoDB")</code></pre> 
</div> 
<ul> 
 <li>Deploy the Lambda function.</li> 
 <li>On the <strong>Configuration </strong>tab, choose <strong>Edit</strong> to edit the trigger.</li> 
</ul> 
<p><img alt="edit trigger" class="alignnone size-full wp-image-47275" height="488" src="https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2024/03/19/bdb3981-10.png" style="margin: 10px 0px 10px 0px;" width="1301" /></p> 
<ul> 
 <li>Select the trigger, then choose <strong>Save</strong>.</li> 
 <li>On the DynamoDB console, choose <strong>Explore items</strong> in the navigation pane.</li> 
 <li>Select the table <code>device_status</code>.</li> 
</ul> 
<p>You will see Lambda is writing events generated in the Kafka topic to DynamoDB.</p> 
<p><img alt="ddb table" class="alignnone size-full wp-image-47276" height="554" src="https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2024/03/19/bdb3981-11.png" style="margin: 10px 0px 10px 0px;" width="1429" /></p> 
<h2>Summary</h2> 
<p>Streaming data pipelines are critical for building real-time applications. However, setting up and managing the infrastructure can be daunting. In this post, we walked through how to build a serverless streaming pipeline on AWS using Amazon MSK, Lambda, DynamoDB, Amazon Data Firehose, and other services. The key benefits are no servers to manage, automatic scalability of the infrastructure, and a pay-as-you-go model using fully managed services.</p> 
<p>Ready to build your own real-time pipeline? Get started today with a free AWS account. With the power of serverless, you can focus on your application logic while AWS handles the undifferentiated heavy lifting. Let’s build something awesome on AWS!</p> 
<h3></h3> 
<hr /> 
<h3>About the Authors</h3> 
<p style="clear: both;"><img alt="" class="alignleft size-full wp-image-47279" height="105" src="https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2024/03/19/masudurs2024.jpg" width="100" /><strong>Masudur Rahaman Sayem</strong> is a Streaming Data Architect at AWS. He works with AWS customers globally to design and build data streaming architectures to solve real-world business problems. He specializes in optimizing solutions that use streaming data services and NoSQL. Sayem is very passionate about distributed computing.</p> 
<p style="clear: both;"><img alt="" class="alignleft size-full wp-image-47280" height="128" src="https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2024/03/19/michael2024.jpg" width="100" /><strong>Michael Oguike</strong> is a Product Manager for Amazon MSK. He is passionate about using data to uncover insights that drive action. He enjoys helping customers from a wide range of industries improve their businesses using data streaming. Michael also loves learning about behavioral science and psychology from books and podcasts.</p>
