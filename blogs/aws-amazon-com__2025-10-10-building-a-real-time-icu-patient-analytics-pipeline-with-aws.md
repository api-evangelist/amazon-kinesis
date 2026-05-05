---
title: "Building a real-time ICU patient analytics pipeline with AWS Lambda event source mapping"
url: "https://aws.amazon.com/blogs/big-data/building-a-real-time-icu-patient-analytics-pipeline-with-aws-lambda-event-source-mapping/"
date: "Fri, 10 Oct 2025 21:55:53 +0000"
author: "Priyanka Chaudhary"
feed_url: "https://aws.amazon.com/blogs/big-data/category/analytics/amazon-kinesis/feed/"
---
<p>In hospital intensive care units (ICUs), continuous patient monitoring is critical. Medical devices generate vast amounts of real-time data on vital signs such as heart rate, blood pressure, and oxygen saturation. The key challenge lies in early detection of patient deterioration through vital sign trending. Healthcare teams must process thousands of data points daily per patient to identify concerning patterns, a task crucial for timely intervention and potentially life-saving care.</p> 
<p><a href="https://docs.aws.amazon.com/lambda/latest/dg/invocation-eventsourcemapping.html" rel="noopener noreferrer" target="_blank">AWS Lambda event source mapping</a> can help in this scenario by automatically polling data streams and triggering functions in real-time without additional infrastructure management. By using <a href="https://aws.amazon.com/lambda/" rel="noopener noreferrer" target="_blank">AWS Lambda</a> for real-time processing of sensor data and storing aggregated results in secure data structures designed for large analytic datasets called Iceberg tables in <a href="https://aws.amazon.com/s3/" rel="noopener noreferrer" target="_blank">Amazon Simple Storage Service (Amazon S3)</a> buckets, medical teams can achieve both immediate alerting capabilities and gain long-term analytical insights, enhancing their ability to provide timely and effective care.</p> 
<p>In this post, we demonstrate how to build a serverless architecture that processes real-time ICU patient monitoring data using Lambda event source mapping for immediate alert generation and data aggregation, followed by persistent storage in Amazon S3 with an Iceberg catalog for comprehensive healthcare analytics. The solution demonstrates how to handle high-frequency vital sign data, implement critical threshold monitoring, and create a scalable analytics platform that can grow with your healthcare organization’s needs and help monitor sensor alert fatigue in the ICU.</p> 
<h2>Architecture</h2> 
<p>The following architecture diagram illustrates a real-time ICU patient analytics system.</p> 
<p><img alt="Arch diagram" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/09/25/BDB-4953-1.png" /></p> 
<p>In this architecture, real-time patient monitoring data from hospital ICU sensors is ingested into <a href="https://aws.amazon.com/iot-core" rel="noopener noreferrer" target="_blank">AWS IoT Core</a>, which then streams the data into <a href="https://aws.amazon.com/kinesis/data-streams/" rel="noopener noreferrer" target="_blank">Amazon Kinesis Data Streams</a>. Two Lambda functions consume this streaming data concurrently for different purposes, both using Lambda event source mapping integration with Kinesis Data Streams. The first Lambda function uses the filtering feature of event source mapping to detect critical health events where SpO2(blood oxygen saturation) levels fall below 90%, immediately triggering notifications to caregivers through <a href="https://aws.amazon.com/sns" rel="noopener noreferrer" target="_blank">Amazon Simple Notification Service (Amazon SNS)</a> for rapid response. The second Lambda function employs the tumbling window feature of event source mapping to aggregate sensor data over 10-minute time intervals. This aggregated data is then systematically stored in S3 buckets in Apache Iceberg format for historical analysis and reporting. The entire pipeline operates in a serverless manner, providing scalable, real-time processing of critical healthcare data while maintaining both immediate alerting capabilities and long-term data storage for analytics.</p> 
<p>Amazon S3 data, with its support for Apache Iceberg table format, enables healthcare organizations to efficiently store and query large volumes of time-series patient data. This solution allows for complex analytical queries across historical patient data while maintaining high performance and cost efficiency.</p> 
<h2>Prerequisites</h2> 
<p>To implement the solution provided in this post, you should have the following:</p> 
<ul> 
 <li>An active AWS account</li> 
 <li>IAM permissions to deploy CloudFormation templates and provision AWS resources</li> 
 <li>Python installed on your machine to run the ICU patient sensor data simulator code</li> 
</ul> 
<h2>Deploy a real-time ICU patient analytics pipeline using CloudFormation</h2> 
<p>You use <a href="https://aws.amazon.com/cloudformation/" rel="noopener noreferrer" target="_blank">AWS CloudFormation</a> templates to create the resources for a real-time data analytics pipeline.</p> 
<ol> 
 <li>To get started, Sign in to the console as Account user and select the appropriate Region.</li> 
 <li>Download and launch <a href="https://github.com/aws-samples/sample-aws-blog-icu-realtime-analytics-pipeline/blob/main/cfn/iot-kinesis-lambda.yaml">CloudFormation template&nbsp;</a> where you want to host the Lambda functions.</li> 
 <li>Choose <strong>Next</strong>.</li> 
 <li>On the <strong>Specify stack details</strong> page, enter a <strong>Stack name</strong> (for example, IoTHealthMonitoring).</li> 
 <li>For <strong>Parameters</strong>, enter the following: 
  <ol type="a"> 
   <li><strong>IoTTopic</strong>: Enter the MQTT topic for your IoT devices (for example, <code>icu/sensors</code>).</li> 
   <li><strong>EmailAddress</strong>: Enter an email address for receiving notifications.</li> 
  </ol> </li> 
 <li>Wait for the stack creation to complete. This process might take 5-10 minutes.</li> 
 <li>After the CloudFormation stack completes, it creates following resources: 
  <ol type="a"> 
   <li>An <a href="https://docs.aws.amazon.com/iot/latest/developerguide/iot-rules.html" rel="noopener noreferrer" target="_blank">AWS IoT Core rule</a> to capture data from the specified IoTTopic topic and routes it to Kinesis data stream.</li> 
   <li>A Kinesis data stream for ingesting IoT sensor data.</li> 
   <li>Two Lambda functions: 
    <ul> 
     <li><code>FilterSensorData</code>: Monitors critical health metrics and sends alerts.</li> 
     <li><code>AggregateSensorData</code>: Aggregates sensor data in 10 minutes window.</li> 
    </ul> </li> 
   <li>An <a href="https://aws.amazon.com/dynamodb" rel="noopener noreferrer" target="_blank">Amazon DynamoDB</a> table (<code>NotificationTimestamps</code>) to store notification timestamps for rate limiting alerts.</li> 
   <li>An Amazon SNS topic and subscription to send email notifications for critical patient conditions.</li> 
   <li>An <a href="https://aws.amazon.com/firehose" rel="noopener noreferrer" target="_blank">Amazon Data Firehose</a> delivery stream to deliver processed data to Amazon S3 using Iceberg format.</li> 
   <li>Amazon S3 buckets to store sensor data.</li> 
   <li><a href="https://aws.amazon.com/athena" rel="noopener noreferrer" target="_blank">Amazon Athena</a> and <a href="https://aws.amazon.com/glue" rel="noopener noreferrer" target="_blank">AWS Glue</a> resources for the database and an Iceberg table for querying aggregated data.</li> 
   <li><a href="https://aws.amazon.com/iam/" rel="noopener noreferrer" target="_blank">AWS Identity and Access Management (IAM)</a> roles and policies to support required permissions for Amazon IoT rules, Lambda functions, and <a href="https://aws.amazon.com/kinesis/data-firehose" rel="noopener noreferrer" target="_blank">Data Firehose</a> streams.</li> 
   <li><a href="https://aws.amazon.com/cloudwatch" rel="noopener noreferrer" target="_blank">Amazon CloudWatch</a> log groups to record for Kinesis Firehose activity and Lambda functions.</li> 
  </ol> </li> 
</ol> 
<h2>Solution walkthrough</h2> 
<p>Now that you’ve deployed the solution, let’s review a functional walkthrough. First, simulate patient vital signs data and send it to AWS IoT Core using the following Python code on your local machine. To run this code successfully, ensure you have the necessary IAM permissions to publish messages to the IoT topic in the AWS account where the solution is deployed.</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">import boto3
import json
import random
import time
# AWS IoT Data client
iot_data_client = boto3.client(
    'iot-data',
    region_name='us-west-2'
)
# IOT Topic to publish
topic = 'icu/sensors'
# Fixed set of patient IDs
patient_ids = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
print("Infinite sensor data simulation...")
try:
    while True:
        for patient_id in patient_ids:
            # Generate sensor data
            message = {
                "patient_id": patient_id,
                "timestamp": int(time.time()),
                "spo2": random.randint(91, 99),
                "heart_rate": random.randint(60, 100),
                "temperature_f": round(random.uniform(97.0, 100.0), 1)
            }
            # Publish to topic
            response = iot_data_client.publish(
                topic=topic,
                qos=1,
                payload=json.dumps(message)
            )
            print(f"Published: {message}")
        # Wait 30 seconds before next round
        print("Sleeping for 30 seconds...\n")
        time.sleep(30)
except KeyboardInterrupt:
    print("\nSimulation stopped by user.")</code></pre> 
</div> 
<p>The following is the format of a sample ICU sensor message produced by the simulator.</p> 
<div class="hide-language"> 
 <pre><code class="lang-json">{
    "patient_id": 1,
    "timestamp": 1683000000,
    "spo2": 85,
    "heart_rate": 75,
    "temperature_f": 98.6
}</code></pre> 
</div> 
<p>Data is published to the <code>icu/sensors</code> IoT topic every 30 seconds for 10 different patients, creating a continuous stream of ICU patient monitoring data. Messages published to AWS IoT Core are passed to Kinesis Data Streams using the following message routing rule deployed by our solution.</p> 
<p><img src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/09/25/BDB-4953-3.png" /></p> 
<p>Two Lambda functions consume data from Data Streams concurrently, both using the Lambda event source mapping integration with Kinesis Data Streams.</p> 
<h3>Event source mapping</h3> 
<p>Lambda event source mapping automatically triggers Lambda functions in response to data changes from supported event sources like <a href="https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Streams.html" rel="noopener noreferrer" target="_blank">Amazon DynamoDB Streams</a>, <a href="https://aws.amazon.com/kinesis/data-streams/" rel="noopener noreferrer" target="_blank">Amazon Kinesis Data Streams</a>, <a href="https://aws.amazon.com/sqs/" rel="noopener noreferrer" target="_blank">Amazon Simple Queue Service (Amazon SQS)</a>, <a href="https://aws.amazon.com/amazon-mq/" rel="noopener noreferrer" target="_blank">Amazon MQ</a>, and <a href="https://aws.amazon.com/msk/" rel="noopener noreferrer" target="_blank">Amazon Managed Streaming for Apache Kafka</a>. This serverless integration works by having Lambda poll these sources for new records, which are then processed in configurable batch sizes ranging from 1 to 10,000 records. When new data is detected, Lambda automatically invokes the function synchronously, handling the scaling automatically based on the workload. The service supports at-least-once delivery and provides robust error handling through retry policies and dead-letter queues for failed events. Event source mappings can be fine-tuned through <a href="https://docs.aws.amazon.com/lambda/latest/dg/services-kinesis-parameters.html" rel="noopener noreferrer" target="_blank">various parameters</a> such as batch windows, maximum record age, and retry attempts, making them highly adaptable to different use cases. This feature is particularly valuable in event-driven architectures, so that customers can focus on business logic while AWS manages the complexities of event processing, scaling, and reliability.</p> 
<p>Event source mapping uses tumbling windows and filtering to process and analyze data.</p> 
<h3>Tumbling windows</h3> 
<p>Tumbling windows in Lambda event processing enable data aggregation in fixed, non-overlapping time intervals, where each event belongs to exactly one window. This is ideal for time-based analytics and periodic reporting. When combined with event source mapping, this approach allows efficient batch processing of events within defined time periods (for example, 10-minute windows), enabling calculations such as average vital signs or cumulative fluid intake and output while optimizing function invocations and resource usage.</p> 
<p>When you configure an event source mapping between Kinesis Data Streams and a Lambda function, use the <strong>Tumbling Window Duration</strong> setting, which appears in the trigger configuration in the Lambda console. The solution you deployed using the CloudFormation template includes the <code>AggregateSensorData</code> Lambda function, which uses a 10-minute tumbling window configuration. Depending on the volume of messages flowing through the Amazon Kinesis stream, the <code>AggregateSensorData</code> function can be invoked multiple times for each 10-minute window, sequentially, with the following attributes in the event supplied to the function.</p> 
<ul> 
 <li><strong>Window start and end:</strong> The beginning and ending timestamps for the current tumbling window.</li> 
 <li><strong>State:</strong> An object containing the state returned from the previous window, which is initially empty. The state object can contain up to 1 MB of data.</li> 
 <li><strong>isFinalInvokeForWindow:</strong>&nbsp;Indicates if this is the last invocation for the tumbling window. This only occurs once per window period.</li> 
 <li><strong>isWindowTerminatedEarly</strong>: A window ends early only if the state exceeds the maximum allowed size of 1 MB.</li> 
</ul> 
<p>In a tumbling window, there is a series of Lambda invocations in the following pattern:</p> 
<p><img src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/09/25/BDB-4953-4.png" /></p> 
<p><code>AggregateSensorData</code> Lambda code snippet:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">def handler(event, context):
    
    state_across_window = event['state']
    # Iterate through each record and decode the base64 data
    for record in event['Records']:
        encoded_data = record['kinesis']['data']
        partition_key = record['kinesis']['partitionKey']
        decoded_bytes = base64.b64decode(encoded_data)
        decoded_str = decoded_bytes.decode('utf-8')
        decoded_json = json.loads(decoded_str)
        # create partition_key attribute if it do not exists in state
        if partition_key not in state_across_window:
            state_across_window[partition_key] = {"min_spo2": decoded_json['spo2'], "max_spo2": decoded_json['spo2'], "avg_spo2": decoded_json['spo2'], "sum_spo2": decoded_json['spo2'], "min_heart_rate": decoded_json['heart_rate'], "max_heart_rate": decoded_json['heart_rate'], "avg_heart_rate": decoded_json['heart_rate'], "sum_heart_rate": decoded_json['heart_rate'], "min_temperature_f": decoded_json['temperature_f'], "max_temperature_f": decoded_json['temperature_f'], "avg_temperature_f": decoded_json['temperature_f'], "sum_temperature_f": decoded_json['temperature_f'], "record_count": 1}
        else:
            min_spo2 = state_across_window[partition_key]['min_spo2'] if state_across_window[partition_key]['min_spo2'] &lt; decoded_json['spo2'] else decoded_json['spo2']
            max_spo2 = state_across_window[partition_key]['max_spo2'] if state_across_window[partition_key]['max_spo2'] &gt; decoded_json['spo2'] else decoded_json['spo2']
            sum_spo2 = state_across_window[partition_key]['sum_spo2'] + decoded_json['spo2']
            min_heart_rate = state_across_window[partition_key]['min_heart_rate'] if state_across_window[partition_key]['min_heart_rate'] &lt; decoded_json['heart_rate'] else decoded_json['heart_rate']
            max_heart_rate = state_across_window[partition_key]['max_heart_rate'] if state_across_window[partition_key]['max_heart_rate'] &gt; decoded_json['heart_rate'] else decoded_json['heart_rate']
            sum_heart_rate = state_across_window[partition_key]['sum_heart_rate'] + decoded_json['heart_rate']
            
            min_temperature_f = state_across_window[partition_key]['min_temperature_f'] if state_across_window[partition_key]['min_temperature_f'] &lt; decoded_json['temperature_f'] else decoded_json['temperature_f']
            max_temperature_f = state_across_window[partition_key]['max_temperature_f'] if state_across_window[partition_key]['max_temperature_f'] &gt; decoded_json['temperature_f'] else decoded_json['temperature_f']
            sum_temperature_f = state_across_window[partition_key]['sum_temperature_f'] + decoded_json['temperature_f']
            
            record_count = state_across_window[partition_key]['record_count'] + 1
            avg_spo2 = sum_spo2/record_count
            avg_heart_rate = sum_heart_rate/record_count
            avg_temperature_f = sum_temperature_f/record_count
            
            state_across_window[partition_key] = {"min_spo2": min_spo2, "max_spo2": max_spo2, "avg_spo2": avg_spo2, "sum_spo2": sum_spo2, "min_heart_rate": min_heart_rate, "max_heart_rate": max_heart_rate, "avg_heart_rate": avg_heart_rate, "sum_heart_rate": sum_heart_rate, "min_temperature_f": min_temperature_f, "max_temperature_f": max_temperature_f, "avg_temperature_f": avg_temperature_f, "sum_temperature_f": sum_temperature_f, "record_count": record_count}
        
    # Determine if the window is final (window end)
    is_final_window = event.get('isFinalInvokeForWindow', False)
    # Determine if the window is terminated (window ended early)
    is_terminated_window = event.get('isWindowTerminatedEarly', False)
    window_start = event['window']['start']
    window_end = event['window']['end']
    if is_final_window or is_terminated_window:
        firehose_client = boto3.client('firehose')
        firehose_stream = os.environ['FIREHOSE_STREAM_NAME']
        for key, value in state_across_window.items():
            value['patient_id'] = key
            value['window_start'] = window_start
            value['window_end'] = window_end
            
            firehose_client.put_record(
                DeliveryStreamName= firehose_stream,
                Record={'Data': json.dumps(value) }
            )
        
        return {
            "state": {},
            "batchItemFailures": []
        }
    else:
        print(f"interim call for window: ws: {window_start} we: {window_end}")
        return {
            "state": state_across_window,
            "batchItemFailures": []
        }</code></pre> 
</div> 
<ul> 
 <li>The first invocation contains an empty state object in the event. The function returns a state object containing custom attributes that are specific to the custom logic in the aggregation.</li> 
 <li>The second invocation contains the state object provided by the first Lambda invocation. This function returns an updated state object with new aggregated values. Subsequent invocations follow this same sequence. Following is a sample of the aggregated state, which can be supplied to subsequent Lambda invocations within the same 10-minute tumbling window.</li> 
</ul> 
<div class="hide-language"> 
 <pre><code class="lang-json">{
    "min_spo2": 88,
    "max_spo2": 90,
    "avg_spo2": 89.2,
    "sum_spo2": 625,
    "min_heart_rate": 21,
    "max_heart_rate": 22,
    "avg_heart_rate": 21.1,
    "sum_heart_rate": 148,
    "min_temperature_f": 90,
    "max_temperature_f": 91,
    "avg_temperature_f": 90.1,
    "sum_temperature_f": 631,
    "record_count": 7,
    "patient_id": "44",
    "window_start": "2025-05-29T20:51:00Z",
    "window_end": "2025-05-29T20:52:00Z"
}</code></pre> 
</div> 
<ul> 
 <li>The final invocation in the tumbling window has the&nbsp;<code>isFinalInvokeForWindow</code>&nbsp;flag set to the true. This contains the state returned by the most recent Lambda invocation. This invocation is responsible for passing aggregated state messages to the Data Firehose stream, which delivers data to the Amazon S3 bucket using Iceberg data format.</li> 
 <li>After the aggregated data is sent to Amazon S3, you can query the data using Athena.</li> 
</ul> 
<div class="hide-language"> 
 <pre><code class="lang-sql">Query: SELECT * FROM "cfdb_&lt;&lt;Database&gt;&gt;"."table_&lt;&lt;Table&gt;&gt;"</code></pre> 
</div> 
<p>Sample result of the preceding Athena query:</p> 
<p><img src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/09/25/BDB-4953-5.png" /></p> 
<h3>Event source mapping with filtering</h3> 
<p>Lambda event source mapping with filtering optimizes data processing from sources like <a href="https://aws.amazon.com/kinesis" rel="noopener noreferrer" target="_blank">Amazon Kinesis</a> by applying JSON pattern filtering before function invocation. This is demonstrated in the ICU patient monitoring solution, where the system filters for SpO2 readings from Kinesis Data Streams that are below 90%. Instead of processing all incoming data, the filtering capability is used to selectively processes only critical readings, significantly reducing costs and processing overhead. The solution uses DynamoDB for sophisticated state management, tracking low SpO2 events through a schema combining <code>PatientID</code> and timestamp-based keys within defined monitoring windows.</p> 
<p>This state-aware implementation balances clinical urgency with operational efficiency by sending immediate Amazon SNS notifications when critical conditions are first detected while implementing a 15-minute alert suppression window to prevent alert fatigue among healthcare providers. By maintaining state across multiple Lambda invocations, the system helps ensure rapid response to potentially life-threatening situations while minimizing unnecessary notifications for the same patient condition. The integration of Lambda’event filtering, DynamoDB state management, and reliable alert delivery provided by Amazon SNS creates a robust, scalable healthcare monitoring solution that exemplifies how AWS services can be strategically combined to address complex requirements while balancing technical efficiency with clinical effectiveness.</p> 
<p><img src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/09/25/BDB-4953-6.png" /></p> 
<p>Filter sensor data Lambda code snippet:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">sns_client = boto3.client('sns')
dynamodb = boto3.resource('dynamodb')
table_name = os.environ['DYNAMODB_TABLE']
sns_topic_arn = os.environ['SNS_TOPIC_ARN']
table = dynamodb.Table(table_name)
FIFTEEN_MINUTES = 15 * 60  # 15 minutes in seconds
def handler(event, context):
    for record in event['Records']:
        print(f"Aggregated event: {record}")
        encoded_data = record['kinesis']['data']
        partition_key = record['kinesis']['partitionKey']
        decoded_bytes = base64.b64decode(encoded_data)
        decoded_str = decoded_bytes.decode('utf-8')
        # Check last notification timestamp from DynamoDB
        try:
            response = table.get_item(Key={'partition_key': partition_key})
            item = response.get('Item')
            now = int(time.time())
            if item:
                last_sent = item.get('timestamp', 0)
                if now - last_sent &lt; FIFTEEN_MINUTES:
                    print(f"Notification for {partition_key} skipped (sent recently)")
                    continue
            # Send SNS Notification
            sns_response = sns_client.publish(
                TopicArn=sns_topic_arn,
                Message=f"Patient SpO2 below 90 percentage event information: {decoded_str}",
                Subject=f"Low SpO2 detected for patient ID {partition_key}"
            )
            print("Message sent to SNS! MessageId:", sns_response['MessageId'])
            # Update DynamoDB with current timestamp and TTL
            table.put_item(Item={
                'partition_key': partition_key,
                'timestamp': now,
                'ttl': now + FIFTEEN_MINUTES + 60  # Add extra buffer to TTL
            })
        except Exception as e:
            print("Error processing event:", e)
            return {
                'statusCode': 500,
                'body': json.dumps('Error processing event')
            }
    return {
        'statusCode': 200,
        'body': {}
    }</code></pre> 
</div> 
<p>To generate an alert notification through the deployed solution, update the preceding simulator code by setting the SpO2 value to less than 90 and run it again. Within 1 minute, you should receive an alert notification at the email address you provided during stack creation. The following image is an example of an alert notification generated by the deployed solution.</p> 
<p><img alt="" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/09/25/BDB-4953-7.png" /></p> 
<h2>Clean up</h2> 
<p>To avoid ongoing costs after completing this tutorial, <a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-delete-stack.html" rel="noopener noreferrer" target="_blank">delete the CloudFormation stack</a> that you deployed earlier in this post. This will remove most of the AWS resources created for this solution. You might need to <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/empty-bucket.html" rel="noopener noreferrer" target="_blank">manually delete objects</a> created in Amazon S3, because CloudFormation won’t remove non-empty buckets during stack deletion.</p> 
<h2>Conclusion</h2> 
<p>As demonstrated in this post, you can build a serverless real-time analytics pipeline for healthcare monitoring by using AWS IoT Core, Amazon S3 buckets with iceberg format, and Amazon Kinesis Data Streams integration with AWS Lambda event source mapping. This architectural approach eliminates the need for complex code while enabling rapid critical patient care alerts and data aggregation for analysis using Lambda. The solution is particularly valuable for healthcare organizations looking to modernize their patient monitoring systems with real-time capabilities. The architecture can be extended to handle various medical devices and sensor data streams, making it adaptable for different healthcare monitoring scenarios. This post presents one implementation approach, and organizations adopting this solution should ensure the architecture and code meets their specific application performance, security, privacy, and regulatory compliance needs.</p> 
<p>If this post helps you or inspires you to solve a problem, we would love to hear about it!</p> 
<hr /> 
<h3>About the authors</h3> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Nihar Sheth" class="aligncenter size-full wp-image-29797" height="160" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/09/24/icu-4953-author1.png" width="120" />
  </div> 
  <h3 class="lb-h4">Nihar Sheth</h3> 
  <p><a href="https://www.linkedin.com/in/niharsheth-aws/" rel="noopener" target="_blank">Nihar</a> is a Senior Product Manager on the AWS Lambda team at Amazon Web Services. He is passionate about developing intuitive product experiences that solve complex customer problems and enable customers to achieve their business goals.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Pratik Patel" class="aligncenter size-full wp-image-29797" height="160" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/09/24/icu-4953-author2.jpeg" width="120" />
  </div> 
  <h3 class="lb-h4">Pratik Patel</h3> 
  <p><a href="https://www.linkedin.com/in/pratikpatel-aws/" rel="noopener" target="_blank">Pratik</a> is Sr Technical Account Manager and streaming analytics specialist. He works with AWS customers and provides ongoing support and technical guidance to help plan and build solutions using best practices and proactively helps in keeping customers’ AWS environments operationally healthy.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Priyanka Chaudhary" class="aligncenter size-full wp-image-29797" height="160" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/09/24/icu-4953-author3.png" width="120" />
  </div> 
  <h3 class="lb-h4">Priyanka Chaudhary</h3> 
  <p><a href="https://www.linkedin.com/in/priyanka-chaudhary-36365b38/" rel="noopener" target="_blank">Priyanka</a> is Senior Solutions Architect at AWS. She is specialized in data lake and analytics services and helps many customers in this area. As a Solutions Architect, she plays a crucial role in guiding strategic customers through their cloud journey by designing scalable and secure cloud solutions. Outside of work, she loves spending time with friends and family, watching movies, and traveling.</p> 
 </div> 
</footer>
