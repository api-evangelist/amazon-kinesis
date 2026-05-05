---
title: "Gain insights from historical location data using Amazon Location Service and AWS analytics services"
url: "https://aws.amazon.com/blogs/big-data/gain-insights-from-historical-location-data-using-amazon-location-service-and-aws-analytics-services/"
date: "Wed, 13 Mar 2024 16:56:53 +0000"
author: "Alan Peaty"
feed_url: "https://aws.amazon.com/blogs/big-data/category/analytics/amazon-kinesis/kinesis-data-firehose/feed/"
---
<p>Many organizations around the world rely on the use of physical assets, such as vehicles, to deliver a service to their end-customers. By tracking these assets in real time and storing the results, asset owners can derive valuable insights on how their assets are being used to continuously deliver business improvements and plan for future changes. For example, a delivery company operating a fleet of vehicles may need to ascertain the impact from local policy changes outside of their control, such as the announced expansion of an <a href="https://en.wikipedia.org/wiki/Ultra_Low_Emission_Zone" rel="noopener" target="_blank">Ultra-Low Emission Zone (ULEZ)</a>. By combining historical vehicle location data with information from other sources, the company can devise empirical approaches for better decision-making. For example, the company’s procurement team can use this information to make decisions about which vehicles to prioritize for replacement before policy changes go into effect.</p> 
<p>Developers can use the support in <a href="https://aws.amazon.com/location/" rel="noopener" target="_blank">Amazon Location Service</a> for <a href="https://aws.amazon.com/about-aws/whats-new/2023/07/amazon-location-service-device-updates-eventbridge/" rel="noopener" target="_blank">publishing device position updates</a> to <a href="https://aws.amazon.com/eventbridge/" rel="noopener" target="_blank">Amazon EventBridge</a> to build a near-real-time data pipeline that stores locations of tracked assets in <a href="https://aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon Simple Storage Service</a> (Amazon S3). Additionally, you can use <a href="https://aws.amazon.com/lambda/" rel="noopener" target="_blank">AWS Lambda</a> to enrich incoming location data with data from other sources, such as an <a href="https://aws.amazon.com/dynamodb/" rel="noopener" target="_blank">Amazon DynamoDB</a> table containing vehicle maintenance details. Then a data analyst can use the <a href="https://docs.aws.amazon.com/athena/latest/ug/querying-geospatial-data.html" rel="noopener" target="_blank">geospatial querying capabilities</a> of <a href="https://aws.amazon.com/athena/" rel="noopener" target="_blank">Amazon Athena</a> to gain insights, such as the number of days their vehicles have operated in the proposed boundaries of an expanded ULEZ. Because vehicles that do not meet ULEZ emissions standards are subjected to a daily charge to operate within the zone, you can use the location data, along with maintenance data such as age of the vehicle, current mileage, and current emissions standards to estimate the amount the company would have to spend on daily fees.</p> 
<p>This post shows how you can use Amazon Location, EventBridge, Lambda, <a href="https://aws.amazon.com/kinesis/data-firehose/" rel="noopener" target="_blank">Amazon Data Firehose</a>, and Amazon S3 to build a location-aware data pipeline, and use this data to drive meaningful insights using <a href="https://aws.amazon.com/glue/" rel="noopener" target="_blank">AWS Glue</a> and Athena.</p> 
<h2>Overview of solution</h2> 
<p>This is a fully serverless solution for location-based asset management. The solution consists of the following interfaces:</p> 
<ul> 
 <li><strong>IoT or mobile application</strong> – A mobile application or an Internet of Things (IoT) device allows the tracking of a company vehicle while it is in use and transmits its current location securely to the data ingestion layer in AWS. The ingestion approach is not in scope of this post. Instead, a Lambda function in our solution simulates sample vehicle journeys and directly updates Amazon Location tracker objects with randomized locations.</li> 
 <li><strong>Data analytics</strong> – Business analysts gather operational insights from multiple data sources, including the location data collected from the vehicles. Data analysts are looking for answers to questions such as, “How long did a given vehicle historically spend inside a proposed zone, and how much would the fees have cost had the policy been in place over the past 12 months?”</li> 
</ul> 
<p>The following diagram illustrates the solution architecture.<br /> <img alt="Architecture diagram" class="alignnone size-full wp-image-60986" height="100%" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/03/08/BDB-3578_Architecture.png" style="margin: 10px 0px 10px 0px;" width="100%" /></p> 
<p>The workflow consists of the following key steps:</p> 
<ol> 
 <li>The tracking functionality of Amazon Location is used to track the vehicle. Using EventBridge integration, filtered positional updates are published to an EventBridge event bus. This solution uses <a href="https://docs.aws.amazon.com/location/latest/developerguide/start-tracking.html" rel="noopener" target="_blank">distance-based </a>filtering to reduce costs and jitter. Distanced-based filtering ignores location updates in which devices have moved less than 30 meters (98.4 feet).</li> 
 <li>Amazon Location device position events arrive on the EventBridge <code>default</code> bus with <code>source: ["aws.geo"]</code> and <code>detail-type: ["Location Device Position Event"]</code>. One rule is created to forward these events to two downstream targets: a Lambda function, and a Firehose delivery stream.</li> 
 <li>Two different patterns, based on each target, are described in this post to demonstrate different approaches to committing the data to a S3 bucket: 
  <ol type="a"> 
   <li><strong>Lambda function </strong>– The first approach uses a Lambda function to demonstrate how you can use code in the data pipeline to directly transform the incoming location data. You can modify the Lambda function to fetch additional vehicle information from a separate data store (for example, a DynamoDB table or a Customer Relationship Management system) to enrich the data, before storing the results in an S3 bucket. In this model, the Lambda function is invoked for each incoming event.</li> 
   <li><strong>Firehose delivery stream </strong>– The second approach uses a Firehose delivery stream to buffer and batch the incoming positional updates, before storing them in an S3 bucket without modification. This method uses GZIP compression to optimize storage consumption and query performance. You can also use the <a href="https://docs.aws.amazon.com/firehose/latest/dev/data-transformation.html" rel="noopener" target="_blank">data transformation</a> feature of Data Firehose to invoke a Lambda function to perform data transformation in batches.</li> 
  </ol> </li> 
 <li>AWS Glue crawls both S3 bucket paths, populates the AWS Glue database tables based on the inferred schemas, and makes the data available to other analytics applications through the AWS Glue Data Catalog.</li> 
 <li>Athena is used to run geospatial queries on the location data stored in the S3 buckets. The Data Catalog provides metadata that allows analytics applications using Athena to find, read, and process the location data stored in Amazon S3.</li> 
 <li>This solution includes a Lambda function that continuously updates the Amazon Location tracker with simulated location data from fictitious journeys. The Lambda function is triggered at regular intervals using a scheduled EventBridge rule.</li> 
</ol> 
<p>You can test this solution yourself using the <a href="https://github.com/aws-samples/amazon-location-service-data-analytics" rel="noopener" target="_blank">AWS Samples GitHub repository</a>. The repository contains the <a href="https://aws.amazon.com/serverless/sam/" rel="noopener" target="_blank">AWS Serverless Application Model</a> (AWS SAM) template and Lambda code required to try out this solution. Refer to the instructions in the <a href="https://github.com/aws-samples/amazon-location-service-data-analytics/blob/main/README.md" rel="noopener" target="_blank">README</a> file for steps on how to provision and decommission this solution.</p> 
<p>Visual layouts in some screenshots in this post may look different than those on your <a href="http://aws.amazon.com/console" rel="noopener" target="_blank">AWS Management Console</a>.</p> 
<h2>Data generation</h2> 
<p>In this section, we discuss the steps to manually or automatically generate journey data.</p> 
<h3>Manually generate journey data</h3> 
<p>You can manually update device positions using the <a href="http://aws.amazon.com/cli" rel="noopener" target="_blank">AWS Command Line Interface</a> (AWS CLI) command <code>aws location batch-update-device-position</code>. Replace the <code>tracker-name</code>, <code>device-id</code>, <code>Position</code>, and <code>SampleTime</code> values with your own, and make sure that successive updates are more than 30 meters in distance apart to place an event on the <code>default</code> EventBridge event bus:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-code">aws location batch-update-device-position --tracker-name <em>&lt;tracker-name&gt;</em> --updates "[{\"DeviceId\": \"<em>&lt;device-id&gt;</em>\", \"Position\": [<em>&lt;longitude&gt;</em>, <em>&lt;latitude&gt;</em>], \"SampleTime\": \"<em>&lt;YYYY-MM-DDThh:mm:ssZ&gt;</em>\"}]"</code></pre> 
</div> 
<h3>Automatically generate journey data using the simulator</h3> 
<p>The provided <a href="https://aws.amazon.com/cloudformation/" rel="noopener" target="_blank">AWS CloudFormation</a> template deploys an EventBridge scheduled rule and an accompanying Lambda function that simulates tracker updates from vehicles. This rule is enabled by default, and runs at a frequency specified by the <code>SimulationIntervalMinutes</code> CloudFormation parameter. The data generation Lambda function updates the Amazon Location tracker with a randomized position offset from the vehicles’ base locations.</p> 
<p>Vehicle names and base locations are stored in the <a href="https://github.com/aws-samples/amazon-location-service-data-analytics/blob/main/sam/generate-data/vehicles.json" rel="noopener" target="_blank">vehicles.json</a> file. A vehicle’s starting position is reset each day, and base locations have been chosen to give them the ability to drift in and out of the ULEZ on a given day to provide a realistic journey simulation.</p> 
<p>You can <a href="https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-delete-rule.html" rel="noopener" target="_blank">disable the rule</a> temporarily by navigating to the scheduled rule details on the EventBridge console. Alternatively, change the parameter <code>State: ENABLED</code> to <code>State: DISABLED</code> for the scheduled rule resource <code>GenerateDevicePositionsScheduleRule</code> in the <a href="https://github.com/aws-samples/amazon-location-service-data-analytics/blob/main/sam/template.yml" rel="noopener" target="_blank">template.yml</a> file. Rebuild and re-deploy the AWS SAM template for this change to take effect.</p> 
<h2>Location data pipeline approaches</h2> 
<p>The configurations outlined in this section are deployed automatically by the provided AWS SAM template. The information in this section is provided to describe the pertinent parts of the solution.</p> 
<h3>Amazon Location device position events</h3> 
<p>Amazon Location sends device position update events to EventBridge in the following format:</p> 
<div class="hide-language"> 
 <pre><code class="lang-json">{
    "version":"0",
    "id":"<em>&lt;event-id&gt;</em>",
    "detail-type":"Location Device Position Event",
    "source":"aws.geo",
    "account":"<em>&lt;account-number&gt;</em>",
    "time":"<em>&lt;YYYY-MM-DDThh:mm:ssZ&gt;</em>",
    "region":"<em>&lt;region&gt;</em>",
    "resources":[
        "arn:aws:geo:<em>&lt;region&gt;</em>:<em>&lt;account-number&gt;</em>:tracker/<em>&lt;tracker-name&gt;</em>"
    ],
    "detail":{
        "EventType":"UPDATE",
        "TrackerName":"<em>&lt;tracker-name&gt;</em>",
        "DeviceId":"<em>&lt;device-id&gt;</em>",
        "SampleTime":"<em>&lt;YYYY-MM-DDThh:mm:ssZ&gt;</em>",
        "ReceivedTime":"<em>&lt;YYYY-MM-DDThh:mm:ss.sssZ&gt;</em>",
        "Position":[
            &lt;longitude&gt;, 
            &lt;latitude&gt;
	]
    }
}</code></pre> 
</div> 
<p>You can optionally specify an <a href="https://docs.aws.amazon.com/eventbridge/latest/userguide/eb-transform-target-input.html" rel="noopener" target="_blank">input transformation</a> to modify the format and contents of the device position event data before it reaches the target.</p> 
<h3>Data enrichment using Lambda</h3> 
<p>Data enrichment in this pattern is facilitated through the invocation of a Lambda function. In this example, we call this function <code>ProcessDevicePosition</code>, and use a Python runtime. A custom transformation is applied in the EventBridge target definition to receive the event data in the following format:</p> 
<div class="hide-language"> 
 <pre><code class="lang-json">{
    "EventType":<em>&lt;EventType&gt;</em>,
    "TrackerName":<em>&lt;TrackerName&gt;</em>,
    "DeviceId":<em>&lt;DeviceId&gt;</em>,
    "SampleTime":<em>&lt;SampleTime&gt;</em>,
    "ReceivedTime":<em>&lt;ReceivedTime&gt;</em>,
    "Position":[<em>&lt;Longitude&gt;</em>,<em>&lt;Latitude&gt;</em>]
}</code></pre> 
</div> 
<p>You could apply additional transformations, such as the refactoring of <code>Latitude</code> and <code>Longitude</code> data into separate key-value pairs if this is required by the downstream business logic processing the events.</p> 
<p>The following code demonstrates the Python application logic that is run by the <code>ProcessDevicePosition</code> Lambda function. Error handling has been skipped in this code snippet for brevity. The full code is available in the <a href="https://github.com/aws-samples/amazon-location-service-data-analytics/blob/main/sam/process-data/lambda_handler.py" rel="noopener" target="_blank">GitHub repo</a>.</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">import json
import os
import uuid
import boto3

# Import environment variables from Lambda function.
bucket_name = os.environ["S3_BUCKET_NAME"]
bucket_prefix = os.environ["S3_BUCKET_LAMBDA_PREFIX"]

s3 = boto3.client("s3")

def lambda_handler(event, context):
    key = "%s/%s/%s-%s.json" % (bucket_prefix,
                                event["DeviceId"],
                                event["SampleTime"],
                                str(uuid.uuid4())
    body = json.dumps(event, separators=(",", ":"))
    body_encoded = body.encode("utf-8")
    s3.put_object(Bucket=bucket_name, Key=key, Body=body_encoded)
    return {
        "statusCode": 200,
        "body": "success"
    }</code></pre> 
</div> 
<p>The preceding code creates an S3 object for each device position event received by EventBridge. The code uses the <code>DeviceId</code> as a prefix to write the objects to the bucket.</p> 
<p>You can add additional logic to the preceding Lambda function code to enrich the event data using other sources. The example in the <a href="https://github.com/aws-samples/amazon-location-service-data-analytics/blob/main/sam/process-data/lambda_handler.py" rel="noopener" target="_blank">GitHub repo</a> demonstrates enriching the event with data from a DynamoDB vehicle maintenance table.</p> 
<p>In addition to the prerequisite <a href="https://aws.amazon.com/iam/" rel="noopener" target="_blank">AWS Identity and Access Management</a> (IAM) permissions provided by the role <code>AWSBasicLambdaExecutionRole</code>, the <code>ProcessDevicePosition</code> function requires permissions to perform the S3 <code>put_object</code> action and any other actions required by the data enrichment logic. IAM permissions required by the solution are documented in the <a href="https://github.com/aws-samples/amazon-location-service-data-analytics/blob/main/sam/template.yml" rel="noopener" target="_blank">template.yml</a> file.</p> 
<div class="hide-language"> 
 <pre><code class="lang-json">{
    "Version":"2012-10-17",
    "Statement":[
        {
            "Action":[
                "s3:ListBucket"
            ],
            "Resource":[
                "arn:aws:s3:::<em>&lt;S3_BUCKET_NAME&gt;</em>"
            ],
            "Effect":"Allow"
        },
        {
            "Action":[
                "s3:PutObject"
            ],
            "Resource":[
                "arn:aws:s3:::<em>&lt;S3_BUCKET_NAME&gt;</em>/<em>&lt;S3_BUCKET_LAMBDA_PREFIX&gt;</em>/*"
            ],
            "Effect":"Allow"
        }
    ]
}</code></pre> 
</div> 
<h3>Data pipeline using Amazon Data Firehose</h3> 
<p>Complete the following steps to create your Firehose delivery stream:</p> 
<ol> 
 <li>On the Amazon Data Firehose console, choose <strong>Firehose streams </strong>in the navigation pane.</li> 
 <li>Choose <strong>Create Firehose stream</strong>.</li> 
 <li>For <strong>Source</strong>, choose as <strong>Direct PUT</strong>.</li> 
 <li>For <strong>Destination</strong>, choose <strong>Amazon S3</strong>.</li> 
 <li>For <strong>Firehose stream name</strong>, enter a name (for this post, <code>ProcessDevicePositionFirehose</code>).<br /> <img alt="Create Firehose stream" class="alignnone size-full wp-image-60965" height="666" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/03/08/BDB-3578_Image-001.png" style="margin: 10px 0px 10px 0px;" width="816" /></li> 
 <li>Configure the destination settings with details about the S3 bucket in which the location data is stored, along with the partitioning strategy: 
  <ol type="a"> 
   <li>Use <span style="color: #ff0000;"><em>&lt;S3_BUCKET_NAME&gt;</em></span> and <span style="color: #ff0000;"><em>&lt;S3_BUCKET_FIREHOSE_PREFIX&gt;</em></span> to determine the bucket and object prefixes.</li> 
   <li>Use <code>DeviceId</code> as an additional prefix to write the objects to the bucket.</li> 
  </ol> </li> 
 <li>Enable <strong>Dynamic partitioning</strong> and <strong>New line delimiter</strong> to make sure partitioning is automatic based on <code>DeviceId</code>, and that new line delimiters are added between records in objects that are delivered to Amazon S3.</li> 
</ol> 
<p>These are required by AWS Glue to later crawl the data, and for Athena to recognize individual records.<br /> <img alt="Destination settings for Firehose stream" class="alignnone size-full wp-image-60966" height="517" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/03/08/BDB-3578_Image-002.png" style="margin: 10px 0px 10px 0px;" width="1429" /></p> 
<h3>Create an EventBridge rule and attach targets</h3> 
<p>The EventBridge rule <code>ProcessDevicePosition</code> defines two targets: the <code>ProcessDevicePosition</code> Lambda function, and the <code>ProcessDevicePositionFirehose</code> delivery stream. Complete the following steps to create the rule and attach targets:</p> 
<ol> 
 <li>On the EventBridge console, create a new rule.</li> 
 <li>For <strong>Name</strong>, enter a name (for this post, <code>ProcessDevicePosition</code>).</li> 
 <li>For <strong>Event bus</strong>¸ choose <strong>default</strong>.</li> 
 <li>For <strong>Rule type</strong>¸ select <strong>Rule with an event pattern</strong>.<br /> <img alt="EventBridge rule detail" class="alignnone size-full wp-image-60967" height="580" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/03/08/BDB-3578_Image-003.png" style="margin: 10px 0px 10px 0px;" width="821" /></li> 
 <li>For <strong>Event source</strong>, select <strong>AWS events or EventBridge partner events</strong>.<br /> <img alt="EventBridge event source" class="alignnone size-full wp-image-60968" height="287" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/03/08/BDB-3578_Image-004.png" style="margin: 10px 0px 10px 0px;" width="813" /></li> 
 <li>For <strong>Method</strong>, select <strong>Use pattern form</strong>.</li> 
 <li>In the <strong>Event pattern</strong> section, specify <strong>AWS services</strong> as the source, <strong>Amazon Location Service</strong> as the specific service, and <strong>Location Device Position Event </strong>as the event type.<br /> <img alt="EventBridge creation method" class="alignnone size-full wp-image-60969" height="764" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/03/08/BDB-3578_Image-005.png" style="margin: 10px 0px 10px 0px;" width="813" /></li> 
 <li>For<strong> Target 1</strong>, attach the <code>ProcessDevicePosition</code> Lambda function as a target.<br /> <img alt="EventBridge target 1" class="alignnone size-full wp-image-60970" height="485" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/03/08/BDB-3578_Image-006.png" style="margin: 10px 0px 10px 0px;" width="812" /></li> 
 <li>We use <strong>Input transformer</strong> to customize the event that is committed to the S3 bucket.<br /> <img alt="EventBridge target 1 transformer" class="alignnone size-full wp-image-60971" height="741" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/03/08/BDB-3578_Image-007.png" style="margin: 10px 0px 10px 0px;" width="1015" /></li> 
 <li>Configure<strong> Input paths map</strong> and <strong>Input template</strong> to organize the payload into the desired format. 
  <ol type="a"> 
   <li>The following code is the input paths map: 
    <div class="hide-language"> 
     <pre><code class="lang-json">{
    EventType: $.detail.EventType
    TrackerName: $.detail.TrackerName
    DeviceId: $.detail.DeviceId
    SampleTime: $.detail.SampleTime
    ReceivedTime: $.detail.ReceivedTime
    Longitude: $.detail.Position[0]
    Latitude: $.detail.Position[1]
}</code></pre> 
    </div> </li> 
   <li>The following code is the input template: 
    <div class="hide-language"> 
     <pre><code class="lang-json">{
    "EventType":<em>&lt;EventType&gt;</em>,
    "TrackerName":<em>&lt;TrackerName&gt;</em>,
    "DeviceId":<em>&lt;DeviceId&gt;</em>,
    "SampleTime":<em>&lt;SampleTime&gt;</em>,
    "ReceivedTime":<em>&lt;ReceivedTime&gt;</em>,
    "Position":[<em>&lt;Longitude&gt;</em>, <em>&lt;Latitude&gt;</em>]
}</code></pre> 
    </div> </li> 
  </ol> </li> 
 <li>For <strong>Target 2</strong>, choose the <code>ProcessDevicePositionFirehose</code> delivery stream as a target.<br /> <img alt="EventBridge target 2" class="alignnone size-full wp-image-60972" height="626" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/03/09/BDB-3578_Image-008.png" style="margin: 10px 0px 10px 0px;" width="811" /></li> 
</ol> 
<p>This target requires an IAM role that allows one or multiple records to be written to the Firehose delivery stream:</p> 
<div class="hide-language"> 
 <pre><code class="lang-json">{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "firehose:PutRecord",
                "firehose:PutRecords"
            ],
            "Resource": [
                "arn:aws:firehose:<em>&lt;region&gt;</em>:<em>&lt;account-id&gt;</em>:deliverystream/<em>&lt;delivery-stream-name&gt;</em>"
            ],
            "Effect": "Allow"
        }
    ]
}</code></pre> 
</div> 
<h2>Crawl and catalog the data using AWS Glue</h2> 
<p>After sufficient data has been generated, complete the following steps:</p> 
<ol> 
 <li>On the AWS Glue console, choose <strong>Crawlers</strong> in the navigation pane.</li> 
 <li>Select the crawlers that have been created, <code>location-analytics-glue-crawler-lambda</code> and <code>location-analytics-glue-crawler-firehose</code>.</li> 
 <li>Choose <strong>Run</strong>.</li> 
</ol> 
<p>The crawlers will automatically classify the data into JSON format, group the records into tables and partitions, and commit associated metadata to the AWS Glue Data Catalog.<br /> <img alt="Crawlers" class="alignnone size-full wp-image-60973" height="521" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/03/08/BDB-3578_Image-009.png" style="margin: 10px 0px 10px 0px;" width="1635" /></p> 
<ol start="4"> 
 <li>When the <strong>Last run</strong> statuses of both crawlers show as <strong>Succeeded</strong>, confirm that two tables (<code>lambda</code> and <code>firehose</code>) have been created on the <strong>Tables</strong> page.</li> 
</ol> 
<p>The solution partitions the incoming location data based on the <code>deviceid</code> field. Therefore, as long as there are no new devices or schema changes, the crawlers don’t need to run again. However, if new devices are added, or a different field is used for partitioning, the crawlers need to run again.<br /> <img alt="Tables" class="alignnone size-full wp-image-60974" height="418" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/03/08/BDB-3578_Image-010.png" style="margin: 10px 0px 10px 0px;" width="1632" /></p> 
<p>You’re now ready to query the tables using Athena.</p> 
<h2>Query the data using Athena</h2> 
<p>Athena is a serverless, interactive analytics service built to analyze unstructured, semi-structured, and structured data where it is hosted. If this is your first time using the Athena console, <a href="https://docs.aws.amazon.com/athena/latest/ug/getting-started.html" rel="noopener" target="_blank">follow the instructions</a> to set up a query result location in Amazon S3. To query the data with Athena, complete the following steps:</p> 
<ol> 
 <li>On the Athena console, open the query editor.</li> 
 <li>For <strong>Data source</strong>, choose <code>AwsDataCatalog</code>.</li> 
 <li>For <strong>Database</strong>, choose <code>location-analytics-glue-database</code>.</li> 
 <li>On the options menu (three vertical dots), choose <strong>Preview Table </strong>to query the content of both tables.<br /> <img alt="Preview table" class="alignnone size-full wp-image-60975" height="742" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/03/08/BDB-3578_Image-011.png" style="margin: 10px 0px 10px 0px;" width="1628" /></li> 
</ol> 
<p>The query displays 10 sample positional records currently stored in the table. The following screenshot is an example from previewing the <code>firehose</code> table. The <code>firehose</code> table stores raw, unmodified data from the Amazon Location tracker.<br /> <img alt="Query results" class="alignnone size-full wp-image-60976" height="687" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/03/08/BDB-3578_Image-012.png" style="margin: 10px 0px 10px 0px;" width="1179" /><br /> You can now experiment with geospatial queries.The <a href="https://data.london.gov.uk/dataset/ultra_low_emissions_zone_expansion_new" rel="noopener" target="_blank">GeoJSON file for the 2021 London ULEZ expansion</a> is part of the repository, and has already been converted into a query compatible with both Athena tables.</p> 
<ol start="5"> 
 <li>Copy and paste the content from the <a href="https://github.com/aws-samples/amazon-location-service-data-analytics/blob/main/examples/firehose/1-firehose-athena-ulez-2021-create-view.sql" rel="noopener" target="_blank">1-firehose-athena-ulez-2021-create-view.sql</a> file found in the <code>examples/firehose</code> folder into the query editor.</li> 
</ol> 
<p>This query uses the <code>ST_Within</code> geospatial function to determine if a recorded position is inside or outside the ULEZ zone defined by the polygon. A new view called <code>ulezvehicleanalysis_firehose</code> is created with a new column, <code>insidezone</code>, which captures whether the recorded position exists within the zone.</p> 
<p>A simple Python <a href="https://github.com/aws-samples/amazon-location-service-data-analytics/blob/main/examples/utilities/convert_to_st_polygon.py" rel="noopener" target="_blank">utility</a> is provided, which converts the polygon features found in the downloaded GeoJSON file into <code>ST_Polygon</code> strings based on the <a href="https://en.wikipedia.org/wiki/Well-known_text_representation_of_geometry" rel="noopener" target="_blank">well-known text format</a> that can be used directly in an Athena query.</p> 
<ol start="6"> 
 <li>Choose <strong>Preview View</strong> on the <code>ulezvehicleanalysis_firehose</code> view to explore its content.<br /> <img alt="Preview view" class="alignnone size-full wp-image-60977" height="724" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/03/08/BDB-3578_Image-013.png" style="margin: 10px 0px 10px 0px;" width="1552" /></li> 
</ol> 
<p>You can now run queries against this view to gain overarching insights.</p> 
<ol start="7"> 
 <li>Copy and paste the content from the <a href="https://github.com/aws-samples/amazon-location-service-data-analytics/blob/main/examples/firehose/2-firehose-athena-ulez-2021-query-days-in-zone.sql" rel="noopener" target="_blank">2-firehose-athena-ulez-2021-query-days-in-zone.sql</a> file found in the <code>examples/firehose</code> folder into the query editor.</li> 
</ol> 
<p>This query establishes the total number of days each vehicle has entered ULEZ, and what the expected total charges would be. The query has been parameterized using the <code>?</code> placeholder character. <a href="https://docs.aws.amazon.com/athena/latest/ug/querying-with-prepared-statements.html" rel="noopener" target="_blank">Parameterized queries</a> allow you to rerun the same query with different parameter values.</p> 
<ol start="8"> 
 <li>Enter the daily fee amount for <strong>Parameter 1</strong>, then run the query.<br /> <img alt="Query editor" class="alignnone size-full wp-image-60978" height="585" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/03/09/BDB-3578_Image-014.png" style="margin: 10px 0px 10px 0px;" width="1564" /></li> 
</ol> 
<p>The results display each vehicle, the total number of days spent in the proposed ULEZ, and the total charges based on the daily fee you entered.<br /> <img alt="Query results" class="alignnone size-full wp-image-60979" height="503" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/03/08/BDB-3578_Image-015.png" style="margin: 10px 0px 10px 0px;" width="890" /><br /> You can repeat this exercise using the <code>lambda</code> table. Data in the <code>lambda</code> table is augmented with additional vehicle details present in the vehicle maintenance DynamoDB table at the time it is processed by the Lambda function. The solution supports the following fields:</p> 
<ul> 
 <li><code>MeetsEmissionStandards</code> (Boolean)</li> 
 <li><code>Mileage</code> (Number)</li> 
 <li><code>PurchaseDate</code> (String, in <code>YYYY-MM-DD</code> format)</li> 
</ul> 
<p>You can also enrich the new data as it arrives.</p> 
<ol start="9"> 
 <li>On the DynamoDB console, find the vehicle maintenance table under <strong>Tables</strong>. The table name is provided as output <code>VehicleMaintenanceDynamoTable</code> in the deployed CloudFormation stack.</li> 
 <li>Choose <strong>Explore table items</strong> to view the content of the table.</li> 
 <li>Choose <strong>Create item</strong> to create a new record for a vehicle.<br /> <img alt="Create item" class="alignnone size-full wp-image-60980" height="289" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/03/08/BDB-3578_Image-016.png" style="margin: 10px 0px 10px 0px;" width="1010" /></li> 
 <li>Enter <code>DeviceId</code> (such as <code>vehicle1</code> as a String), <code>PurchaseDate</code> (such as <code>2005-10-01</code> as a String), <code>Mileage</code> (such as <code>10000</code> as a Number), and <code>MeetsEmissionStandards</code> (with a value such as <code>False</code> as Boolean).</li> 
 <li>Choose <strong>Create item</strong> to create the record.<br /> <img alt="Create item" class="alignnone size-full wp-image-60981" height="537" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/03/08/BDB-3578_Image-017.png" style="margin: 10px 0px 10px 0px;" width="1340" /></li> 
 <li>Duplicate the newly created record with additional entries for other vehicles (such as for <code>vehicle2</code> or <code>vehicle3</code>), modifying the values of the attributes slightly each time.</li> 
 <li>Rerun the <code>location-analytics-glue-crawler-lambda</code> AWS Glue crawler after new data has been generated to confirm that the update to the schema with new fields is registered.</li> 
 <li>Copy and paste the content from the <a href="https://github.com/aws-samples/amazon-location-service-data-analytics/blob/main/examples/lambda/1-lambda-athena-ulez-2021-create-view.sql" rel="noopener" target="_blank">1-lambda-athena-ulez-2021-create-view.sql</a> file found in the <code>examples/lambda</code> folder into the query editor.</li> 
 <li>Preview the <code>ulezvehicleanalysis_lambda</code> view to confirm that the new columns have been created.</li> 
</ol> 
<p>If errors such as <code>Column 'mileage' cannot be resolved</code> are displayed, the data enrichment is not taking place, or the AWS Glue crawler has not yet detected updates to the schema.</p> 
<p>If the <strong>Preview table option </strong>is only returning results from before you created records in the DynamoDB table, return the query results in descending order using <code>sampletime</code> (for example, <code>order by sampletime desc limit 100;</code>).<br /> <img alt="Query results" class="alignnone size-full wp-image-60982" height="696" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/03/08/BDB-3578_Image-018.png" style="margin: 10px 0px 10px 0px;" width="1178" /><br /> Now we focus on the vehicles that don’t currently meet emissions standards, and order the vehicles in descending order based on the mileage per year (calculated using the latest mileage / age of vehicle in years).</p> 
<ol start="18"> 
 <li>Copy and paste the content from the <a href="https://github.com/aws-samples/amazon-location-service-data-analytics/blob/main/examples/lambda/2-lambda-athena-ulez-2021-query-days-in-zone.sql" rel="noopener" target="_blank">2-lambda-athena-ulez-2021-query-days-in-zone.sql</a> file found in the <code>examples/lambda</code> folder into the query editor.<br /> <img alt="Query results" class="alignnone size-full wp-image-61147" height="552" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/03/11/BDB-3578_Image-019-1.png" style="margin: 10px 0px 10px 0px;" width="1190" /></li> 
</ol> 
<p>In this example, we can see that out of our fleet of vehicles, five have been reported as not meeting emission standards. We can also see the vehicles that have accumulated high mileage per year, and the number of days spent in the proposed ULEZ. The fleet operator may now decide to prioritize these vehicles for replacement. Because location data is enriched with the most up-to-date vehicle maintenance data at the time it is ingested, you can further evolve these queries to run over a defined time window. For example, you could factor in mileage changes within the past year.</p> 
<p>Due to the dynamic nature of the data enrichment, any new data being committed to Amazon S3, along with the query results, will be altered as and when records are updated in the DynamoDB vehicle maintenance table.</p> 
<h2>Clean up</h2> 
<p>Refer to the instructions in the <a href="https://github.com/aws-samples/amazon-location-service-data-analytics/blob/main/README.md" rel="noopener" target="_blank">README</a> file to clean up the resources provisioned for this solution.</p> 
<h2>Conclusion</h2> 
<p>This post demonstrated how you can use Amazon Location, EventBridge, Lambda, Amazon Data Firehose, and Amazon S3 to build a location-aware data pipeline, and use the collected device position data to drive analytical insights using AWS Glue and Athena. By tracking these assets in real time and storing the results, companies can derive valuable insights on how effectively their fleets are being utilized and better react to changes in the future. You can now explore extending this sample code with your own device tracking data and analytics requirements.</p> 
<hr /> 
<h3>About the Authors</h3> 
<p style="clear: both;"><img alt="" class="wp-image-61175 size-full alignleft" height="100" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/03/12/BDB-3578_Author-Alan-100.png" width="100" /><strong>Alan Peaty</strong> is a Senior Partner Solutions Architect at AWS. Alan helps Global Systems Integrators (GSIs) and Global Independent Software Vendors (GISVs) solve complex customer challenges using AWS services. Prior to joining AWS, Alan worked as an architect at systems integrators to translate business requirements into technical solutions. Outside of work, Alan is an IoT enthusiast and a keen runner who loves to hit the muddy trails of the English countryside.</p> 
<p style="clear: both;"><strong><img alt="" class="size-full wp-image-20210 alignleft" height="100" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2021/07/19/Parag-Srivastava-1.jpg" width="100" />Parag Srivastava</strong> is a Solutions Architect at AWS, helping enterprise customers with successful cloud adoption and migration. During his professional career, he has been extensively involved in complex digital transformation projects. He is also passionate about building innovative solutions around geospatial aspects of addresses.</p>
