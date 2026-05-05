---
title: "Optimize industrial IoT analytics with Amazon Data Firehose and Amazon S3 Tables with Apache Iceberg"
url: "https://aws.amazon.com/blogs/big-data/optimize-industrial-iot-analytics-with-amazon-data-firehose-and-amazon-s3-tables-with-apache-iceberg/"
date: "Fri, 25 Jul 2025 16:20:07 +0000"
author: "Ashok Padmanabhan"
feed_url: "https://aws.amazon.com/blogs/big-data/category/analytics/amazon-kinesis/feed/"
---
<p>Manufacturing organizations are racing to digitize their operations through Industry 4.0 initiatives. A key challenge they face is capturing, processing, and analyzing real-time data from industrial equipment to enable data-driven decision making.Modern manufacturing facilities generate massive amounts of real-time data from their production lines. Capturing this valuable data requires a two-tier architecture: first, an edge device that understands industrial protocols collects data directly from the shop floor sensors. Then, these edge gateways securely buffer and transmit the data to AWS Cloud, providing reliability during network interruptions.</p> 
<p>In this post, we show how to use AWS service integrations to minimize custom code while providing a robust platform for industrial data ingestion, processing, and analytics. By using <a href="https://aws.amazon.com/s3/features/tables/" rel="noopener noreferrer" target="_blank">Amazon S3 Tables</a> and its built-in optimizations, you can maximize query performance and minimize costs without additional infrastructure setup. Additionally, <a href="https://aws.amazon.com/greengrass/" rel="noopener noreferrer" target="_blank">AWS IoT Greengrass</a> supports <a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/vpc-interface-endpoints.html" rel="noopener noreferrer" target="_blank">VPC endpoints</a>, and you can <a href="https://aws.amazon.com/blogs/iot/device-onboarding-to-aws-iot-using-virtual-private-cloud-endpoints/" rel="noopener noreferrer" target="_blank">securely</a> communicate between the edge gateway (hosted on premises) and AWS.</p> 
<h2>Solution overview</h2> 
<p>Let’s consider a manufacturing line with and equipment sensors capturing flow rate, temperature, and pressure. To perform analysis on this data, you ingest real-time streaming data from these sensors into the AWS environment using an edge gateway. After data lands in AWS, you can use various analytics services to gain insights.</p> 
<p>To demonstrate the data flow from the edge to the cloud, we have assets, machines, and tools publish data using <a href="https://docs.aws.amazon.com/iot/latest/developerguide/mqtt.html" rel="noopener noreferrer" target="_blank">MQTT</a>. Optionally, we use a simulated edge device that publishes data to a local MQTT endpoint. We use an edge gateway with an AWS IoT Greengrass V2 edge runtime to stream data through <a href="https://docs.aws.amazon.com/firehose/latest/dev/what-is-this-service.html" rel="noopener noreferrer" target="_blank">Amazon Data Firehose</a> in the cloud to S3 Tables.</p> 
<p>The following diagram illustrates the solution architecture.</p> 
<div class="wp-caption aligncenter" id="attachment_80818" style="width: 1034px;">
 <img alt="High Level Arch" class="size-large wp-image-80818" height="297" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/07/14/image-1-18-1024x297.png" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-80818">Fig 1 : High Level Architecture</p>
</div> 
<p>The workflow consists of the following steps:</p> 
<ol> 
 <li>Collect data from Internet of Things (IoT) sensors and stream real-time data from edge devices to the AWS Cloud using AWS IoT Greengrass.</li> 
 <li>Ingest, transform, and land data in near real time using Data Firehose, with the <a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/kinesis-firehose-component.html" rel="noopener noreferrer" target="_blank">Firehose component</a> on AWS IoT Greengrass, and <a href="https://docs.aws.amazon.com/firehose/latest/dev/apache-iceberg-destination.html" rel="noopener noreferrer" target="_blank">S3 Tables integration</a>.</li> 
 <li>Store and organize the tabular data using S3 Tables, which provides purpose-built storage for Apache Iceberg format with a simple, performant, and cost-effective querying solution.</li> 
 <li>Query and analyze the tabular data using <a href="https://docs.aws.amazon.com/athena" rel="noopener noreferrer" target="_blank">Amazon Athena</a>.</li> 
</ol> 
<p>The edge data flow consists of the following key components:</p> 
<ul> 
 <li><strong>IoT device to local MQTT broker</strong> – A simulated device used to generate data for the purposes of this post. In a typical production implementation, this would be your equipment or gateway that supports MQTT. IoT devices can publish messages to a local MQTT broker (Moquette) running on AWS IoT Greengrass.</li> 
 <li><strong>MQTT bridge</strong> – The <a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/mqtt-bridge-component.html" rel="noopener noreferrer" target="_blank">MQTT bridge component</a> relays messages between: 
  <ul> 
   <li>MQTT broker (where client devices communicate)</li> 
   <li>Local AWS IoT Greengrass publish/subscribe (IPC)</li> 
  </ul> </li> 
 <li><strong>Local PubSub (custom) component</strong> – This component completes the following tasks: 
  <ul> 
   <li>Subscribes to the local <a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/ipc-publish-subscribe.html" rel="noopener noreferrer" target="_blank">IPC</a> messages.</li> 
   <li>Forwards messages to the <code>kinesisfirehose/message</code> topic.</li> 
   <li>Uses the IPC interface to subscribe to messages.</li> 
  </ul> </li> 
 <li><strong>Firehose component</strong> – The <a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/kinesis-firehose-component.html" rel="noopener noreferrer" target="_blank">Firehose component</a> subscribes to the <code>kinesisfirehose/message</code> topic. The component then streams the data to Data Firehose in the cloud. It uses QoS 1 for reliable message delivery.</li> 
</ul> 
<p>You can scale this solution to multiple edge locations, so you have a seamless view of data across multiple locations of the manufacturing site, as a low-code solution.In the following sections, we walk through the steps to configure the cloud data ingestion flow:</p> 
<ol> 
 <li>Create an <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-tables.html" rel="noopener noreferrer" target="_blank">S3 Tables</a> bucket and enable integration with AWS analytics services.</li> 
 <li>Create a namespace in the table bucket using the <a href="http://aws.amazon.com/cli" rel="noopener noreferrer" target="_blank">AWS Command Line Interface</a> (AWS CLI).</li> 
 <li>Create a table in the table bucket with the defined schema using the AWS CLI.</li> 
 <li>Create an <a href="https://aws.amazon.com/iam/" rel="noopener noreferrer" target="_blank">AWS Identity and Access Management</a> (IAM) <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html" rel="noopener noreferrer" target="_blank">role</a> for Data Firehose with necessary permissions.</li> 
 <li>Configure <a href="https://aws.amazon.com/lake-formation/" rel="noopener noreferrer" target="_blank">AWS Lake Formation</a> permissions: 
  <ul> 
   <li>Grant Super permissions on specific tables for the Data Firehose role.</li> 
  </ul> </li> 
 <li>Set up a Data Firehose stream: 
  <ul> 
   <li>Choose <strong>Direct PUT</strong> as the source and Iceberg tables as the destination.</li> 
   <li>Configure the destination settings with database and table names.</li> 
   <li>Specify an <a href="http://aws.amazon.com/s3" rel="noopener noreferrer" target="_blank">Amazon Simple Storage Service</a> (Amazon S3) bucket for error output.</li> 
   <li>Associate the IAM role created earlier.</li> 
  </ul> </li> 
 <li>Verify and query data using Athena: 
  <ul> 
   <li>Grant Lake Formation permissions for Athena access.</li> 
   <li>Query the table to verify data ingestion.</li> 
  </ul> </li> 
</ol> 
<h2>Prerequisites</h2> 
<p>You must have the following prerequisites:</p> 
<ul> 
 <li>An AWS account</li> 
 <li>The required IAM privileges to launch AWS IoT Greengrass on an edge gateway (or another <a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/setting-up.html#supported-platforms" rel="noopener noreferrer" target="_blank">supported device</a>)</li> 
 <li>An <a href="http://aws.amazon.com/ec2" rel="noopener noreferrer" target="_blank">Amazon Elastic Compute Cloud</a> (Amazon EC2) instance with a supported operating system to perform a proof of concept</li> 
</ul> 
<h2>Install AWS IoT Greengrass on the edge gateway</h2> 
<p>For instructions to install AWS IoT Greengrass, refer to <a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/install-greengrass-core-v2.html" rel="noopener noreferrer" target="_blank">Install the AWS IoT Greengrass Core software</a>. After you complete the installation, you will have a core device provisioned, as shown in the following screenshot. The status of the device says <strong>Healthy</strong>, which means that your account is able to communicate with the device successfully.</p> 
<p>For a proof of concept, you can use an <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/LaunchingAndUsingInstances.html" rel="noopener noreferrer" target="_blank">Ubuntu-based EC2 instance</a> as your edge gateway.</p> 
<div class="wp-caption aligncenter" id="attachment_80812" style="width: 1034px;">
 <img alt="Greengrass Core Device" class="wp-image-80812 size-large" height="301" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/07/14/image-2-13-1024x301.png" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-80812">Fig 2: Greengrass Core Device</p>
</div> 
<h2>Provision a Data Firehose stream</h2> 
<p>For detailed steps on setting up Data Firehose to deliver data to Iceberg tables, refer to <a href="https://docs.aws.amazon.com/firehose/latest/dev/apache-iceberg-destination.html" rel="noopener noreferrer" target="_blank">Deliver data to Apache Iceberg Tables with Amazon Data Firehose</a>. For S3 Tables integration, refer to <a href="https://aws.amazon.com/blogs/storage/build-a-data-lake-for-streaming-data-with-amazon-s3-tables-and-amazon-data-firehose/" rel="noopener noreferrer" target="_blank">Build a data lake for streaming data with Amazon S3 Tables and Amazon Data Firehose</a>.</p> 
<p>Because you’re using AWS IoT Greengrass to stream data, you can skip the Kinesis Data Generator steps mentioned in these tutorials. The data will instead flow from your edge devices through the Greengrass components to Data Firehose.After you complete these steps, you will have a Firehose stream and S3 Tables bucket, as shown in the following screenshot. Note the Amazon Resource Name (ARN) of the Firehose stream to use in subsequent steps.</p> 
<div class="wp-caption aligncenter" id="attachment_80813" style="width: 1034px;">
 <img alt="Amazon Data Firehose Stream" class="size-large wp-image-80813" height="661" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/07/14/image-3-10-1024x661.png" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-80813">Fig 3: Amazon Data Firehose Stream</p>
</div> 
<h2>Deploy the Greengrass components</h2> 
<p>Complete the following steps to configure and deploy the Greengrass components. For more details, refer to <a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/create-deployments.html" rel="noopener noreferrer" target="_blank">Create deployments</a>.</p> 
<ol> 
 <li>Use the following configuration to enable message routing from local MQTT to the AWS IoT Greengrass PubSub component. Note the topic in the code. This is the MQTT topic where the devices will send the data to.</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-css">{
  "reset": [""],
  "merge": {
    "mqttTopicMapping": {
      "HelloWorldIotCoreMapping": {
        "topic": "clients/#",
        "source": "LocalMqtt",
        "target": "Pubsub"
      }
    }
  }
}</code></pre> 
</div> 
<ol start="2"> 
 <li>Use the following configuration to deploy the Firehose component. Use the Firehose stream ARN that you noted earlier.</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-css">{
"reset": [""],
"merge": {
   "lambdaExecutionParameters": {
     "EnvironmentVariables": {
       "DEFAULT_DELIVERY_STREAM_ARN": "arn:aws:firehose:us-east-1:&lt;&lt;account-id&gt;&gt;:deliverystream/&lt;&lt;stream name&gt;&gt;"
         }
     },
   "containerMode": "NoContainer"
      }
}</code></pre> 
</div> 
<ol start="3"> 
 <li>Use the following configuration to deploy the <a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/legacy-subscription-router-component.html" rel="noopener noreferrer" target="_blank">legacy subscription router component</a> (Note that this is a dependent component to the Firehose component):</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-css">{
"reset": [""],
"merge": {
   "subscriptions": {
      "aws-greengrass-kinesisfirehose": {
          "id": "aws-greengrass-kinesisfirehose",
          "source": "component:aws.greengrass.KinesisFirehose",
          "subject": "kinesisfirehose/message/status",
         "target": "cloud"
                  }
           }
         }
}</code></pre> 
</div> 
<ol start="4"> 
 <li>Create and deploy a custom PubSub component. You can use the following <a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/ipc-publish-subscribe.html" rel="noopener noreferrer" target="_blank">sample code snippet</a> in your preferred language to deploy as a Greengrass component. You can use <a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/greengrass-development-kit-cli.html" rel="noopener noreferrer" target="_blank">gdk</a> to create custom components.</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-css">{
"reset": [""],
"merge": {
   "subscriptions": {
      "aws-greengrass-kinesisfirehose": {
         "id": "aws-greengrass-kinesisfirehose",
        "source": "component:aws.greengrass.KinesisFirehose",
        "subject": "kinesisfirehose/message/status",
          "target": "cloud"
        }
        }
    }
       }</code></pre> 
</div> 
<p>After you deploy the components, you will see them on the <strong>Components</strong> tab of your core device.</p> 
<div class="wp-caption aligncenter" id="attachment_80814" style="width: 780px;">
 <img alt="Greengrass Components" class="size-full wp-image-80814" height="855" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/07/14/image-4-10.png" width="770" />
 <p class="wp-caption-text" id="caption-attachment-80814">Fig 4: AWS IoT Greengrass components</p>
</div> 
<h2>Ingest data</h2> 
<p>In this step, you ingest the data from your device to AWS IoT Greengrass, which will subsequently land in Data Firehose. Complete the following steps:</p> 
<ol> 
 <li>From your edge device that is MQTT aware, or your edge gateway, publish the data to the topic defined earlier ( <code>client/#</code>). For example, we publish the data to the <code>client/devices/telemetry</code> MQTT topic.</li> 
 <li>If you want to do this as a proof of concept, refer to <a href="https://docs.aws.amazon.com/iot/latest/developerguide/creating-a-virtual-thing.html" rel="noopener noreferrer" target="_blank">Create a virtual device with Amazon EC2</a> to create a sample IoT device.</li> 
</ol> 
<p>The following code is a sample payload for our example:</p> 
<div class="hide-language"> 
 <pre><code class="lang-css">PAYLOAD="{
\"device_id\": \"$DEVICE_ID\",
\"timestamp\": \"$TIMESTAMP\",
\"temperature\": $TEMPERATURE,
\"pressure\": $PRESSURE,
\"flow_rate\": $FLOW_RATE,
\"vibration\": $VIBRATION,
\"motor_speed\": $MOTOR_SPEED,
\"status\": \"$STATUS\",
\"battery\": $((RANDOM % 30 + 70 )),
}"</code></pre> 
</div> 
<p>For additional details on how to publish messages from a sample device, refer to <a href="https://catalog.us-east-1.prod.workshops.aws/workshops/7c2b04e7-8051-4c71-bc8b-6d2d7ce32727/en-US/030-provisioning-options/60-just-in-time-provisioning" rel="noopener noreferrer" target="_blank">Just-in-time provisioning</a>.</p> 
<p>The MQTT bridge component will route the payload from the MQTT topic (<code>client/devices/telemetry</code>) to an IPC topic by the same name. The custom component that you deployed earlier will listen to the IPC topic <code>client/devices/telemetry</code> and publish to the IPC topic <code>kinesisfirehose/message</code>. The message must follow the structure described in <a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/kinesis-firehose-component.html#kinesis-firehose-component-input-data" rel="noopener noreferrer" target="_blank">Input data</a>.</p> 
<h2>Validate the data in Athena</h2> 
<p>You can now query the data published from the edge IoT device using Athena. On the Athena console, find the catalog and database that you set up, and run the following query:<code>SELECT * FROM &lt;&lt;database&gt;&gt;."device_telemetry" limit 10;</code>You should see the data displayed as shown in the following screenshot. Note the database and table name that you had defined as part of the “Provision a Data Firehose” stream step.</p> 
<div class="wp-caption aligncenter" id="attachment_80815" style="width: 1034px;">
 <img alt="Validate Data in Athena" class="size-large wp-image-80815" height="797" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/07/14/image-5-8-1024x797.png" width="1024" />
 <p class="wp-caption-text" id="caption-attachment-80815">Fig 5: Validate Data in Athena</p>
</div> 
<h2>Scale out the solution</h2> 
<p>In the preceding sections, we showed how multiple equipments can ingest data into the cloud using a single Greengrass edge gateway device. Because manufacturing locations are distributed in a real-world scenario, you might set up Greengrass devices at other sites and publish the data to the same Firehose stream. This makes sure the data from different sites is landed into a single S3 bucket, is partitioned appropriately (<code>Device_Id</code> in our example), and can be queried seamlessly.</p> 
<h2>Clean up</h2> 
<p>After you validate the results, you can delete the following resources to avoid incurring additional costs:</p> 
<ol> 
 <li><a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/terminating-instances.html" rel="noopener noreferrer" target="_blank">Delete the EC2 Ubuntu instance</a> you created for your proof of concept.</li> 
 <li><a href="https://docs.aws.amazon.com/firehose/latest/APIReference/API_DeleteDeliveryStream.html" rel="noopener noreferrer" target="_blank">Delete the Firehose delivery stream</a> and associated resources.</li> 
 <li><a href="https://docs.aws.amazon.com/athena/latest/ug/drop-table.html" rel="noopener noreferrer" target="_blank">Drop the Athena tables</a> created for querying the data.</li> 
 <li><a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-tables-buckets-delete.html" rel="noopener noreferrer" target="_blank">Delete the S3 Tables bucket</a> you provisioned.</li> 
</ol> 
<h2>Conclusion</h2> 
<p>In this post, we showed how to set up a scalable edge-to-cloud near real-time data ingestion framework using <a href="https://docs.aws.amazon.com/greengrass/v2/developerguide/what-is-iot-greengrass.html" rel="noopener noreferrer" target="_blank">AWS IoT Greengrass</a> and start performing analytics on the data within AWS services using a low-code approach. We demonstrated how to optimize the data storage into Iceberg format with <a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-tables.html" rel="noopener noreferrer" target="_blank">S3 Tables</a>, and transform the streaming data before it lands on the storage layer using <a href="https://docs.aws.amazon.com/firehose/latest/dev/what-is-this-service.html" rel="noopener noreferrer" target="_blank">Data Firehose</a>. We also discussed how you can scale this solution horizontally across multiple manufacturing locations (plants or sites) to create a low-code solution to analyze data in near real time.</p> 
<p>To learn more, refer to the following resources:</p> 
<ul> 
 <li><a href="https://docs.aws.amazon.com/firehose/latest/dev/what-is-this-service.html" rel="noopener noreferrer" target="_blank">Amazon Data Firehose Developer Guide</a></li> 
 <li><a href="https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-tables.html" rel="noopener noreferrer" target="_blank">Working with Amazon S3 Tables and table buckets</a></li> 
 <li><a href="https://aws.amazon.com/blogs/storage/build-a-data-lake-for-streaming-data-with-amazon-s3-tables-and-amazon-data-firehose/" rel="noopener noreferrer" target="_blank">Build a data lake for streaming data with Amazon S3 Tables and Amazon Data Firehose</a></li> 
</ul> 
<h3></h3> 
<hr /> 
<h3>About the authors</h3> 
<p style="clear: both;"><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/07/14/image-6-5.jpeg"><img alt="" class="wp-image-80816 size-thumbnail alignleft" height="137" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/07/14/image-6-5-100x137.jpeg" width="100" /></a> <a href="https://www.linkedin.com/in/joyson-lewis">Joyson Neville Lewis</a> is a Sr. Conversational AI Architect with AWS Professional Services. Joyson worked as a Software/Data engineer before diving into the Conversational AI and Industrial IoT space. He assists AWS customers to materialize their AI visions using Voice Assistant/Chatbot and IoT solutions.</p> 
<p style="clear: both;"><img alt="" class="wp-image-80817 size-thumbnail alignleft" height="100" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/07/14/image-7-5-100x100.jpeg" width="100" /><a href="https://www.linkedin.com/in/anilpraneeth/"><strong>Anil Vure</strong></a> is a Sr. IoT Data Architect with AWS Professional services. Anil has extensive experience building large-scale data platforms and works with manufacturing customers designing high-speed data ingestion systems.</p> 
<p style="clear: both;"><strong><img alt="" class="wp-image-80819 size-thumbnail alignleft" height="129" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/07/14/aSHOK-100x129.jpeg" width="100" />Ashok Padmanabhan</strong> is a Sr. IoT Data Architect with AWS Professional Services. Ashok primarily works with manufacturing and automotive customers to design and build Industry 4.0 solutions.</p>
