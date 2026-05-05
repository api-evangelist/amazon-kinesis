---
title: "Uplevel your data architecture with real- time streaming using Amazon Data Firehose and Snowflake"
url: "https://aws.amazon.com/blogs/big-data/uplevel-your-data-architecture-with-real-time-streaming-using-amazon-data-firehose-and-snowflake/"
date: "Fri, 12 Apr 2024 16:31:36 +0000"
author: "Swapna Bandla"
feed_url: "https://aws.amazon.com/blogs/big-data/category/analytics/amazon-kinesis/kinesis-data-firehose/feed/"
---
<p>Today’s fast-paced world demands timely insights and decisions, which is driving the importance of streaming data. Streaming data refers to data that is continuously generated from a variety of sources. The sources of this data, such as clickstream events, change data capture (CDC), application and service logs, and Internet of Things (IoT) data streams are proliferating. Snowflake offers two options to bring streaming data into its platform: Snowpipe and Snowflake Snowpipe Streaming. Snowpipe is suitable for file ingestion (batching) use cases, such as loading large files from <a href="http://aws.amazon.com/s3">Amazon Simple Storage Service</a> (Amazon S3) to Snowflake. Snowpipe Streaming, a newer feature released in March 2023, is suitable for rowset ingestion (streaming) use cases, such as loading a continuous stream of data from <a href="https://aws.amazon.com/kinesis/">Amazon Kinesis Data Streams</a> or <a href="https://aws.amazon.com/msk/">Amazon Managed Streaming for Apache Kafka</a> (Amazon MSK).</p> 
<p>Before Snowpipe Streaming, AWS customers used Snowpipe for both use cases: file ingestion and rowset ingestion. First, you ingested streaming data to Kinesis Data Streams or Amazon MSK, then used Amazon Data Firehose to aggregate and write streams to Amazon S3, followed by using Snowpipe to load the data into Snowflake. However, this multi-step process can result in delays of up to an hour before data is available for analysis in Snowflake. Moreover, it’s expensive, especially when you have small files that Snowpipe has to upload to the Snowflake customer cluster.</p> 
<p>To solve this issue, Amazon Data Firehose now integrates with Snowpipe Streaming, enabling you to capture, transform, and deliver data streams from Kinesis Data Streams, Amazon MSK, and Firehose Direct PUT to Snowflake in seconds at a low cost. With a few clicks on the Amazon Data Firehose console, you can set up a Firehose stream to deliver data to Snowflake. There are no commitments or upfront investments to use Amazon Data Firehose, and you only pay for the amount of data streamed.</p> 
<p>Some key features of Amazon Data Firehose include:</p> 
<ul> 
 <li><strong>Fully managed serverless service</strong> – You don’t need to manage resources, and Amazon Data Firehose automatically scales to match the throughput of your data source without ongoing administration.</li> 
 <li><strong>Straightforward to use with no code</strong> – You don’t need to write applications.</li> 
 <li><strong>Real-time data delivery</strong> – You can get data to your destinations quickly and efficiently in seconds.</li> 
 <li><strong>Integration with over 20 AWS services</strong> – Seamless integration is available for many AWS services, such as Kinesis Data Streams, Amazon MSK, Amazon VPC Flow Logs, AWS WAF logs, Amazon CloudWatch Logs, Amazon EventBridge, AWS IoT Core, and more.</li> 
 <li><strong>Pay-as-you-go model</strong> – You only pay for the data volume that Amazon Data Firehose processes.</li> 
 <li><strong>Connectivity</strong> – Amazon Data Firehose can connect to public or private subnets in your VPC.</li> 
</ul> 
<p>This post explains how you can bring streaming data from AWS into Snowflake within seconds to perform advanced analytics. We explore common architectures and illustrate how to set up a low-code, serverless, cost-effective solution for low-latency data streaming.</p> 
<h2><strong>Overview of solution</strong></h2> 
<p>The following are the steps to implement the solution to stream data from AWS to Snowflake:</p> 
<ol> 
 <li>Create a Snowflake database, schema, and table.</li> 
 <li>Create a Kinesis data stream.</li> 
 <li>Create a Firehose delivery stream with Kinesis Data Streams as the source and Snowflake as its destination using a secure private link.</li> 
 <li>To test the setup, generate sample stream data from the <a href="https://awslabs.github.io/amazon-kinesis-data-generator/web/producer.html">Amazon Kinesis Data Generator</a> (KDG) with the Firehose delivery stream as the destination.</li> 
 <li>Query the Snowflake table to validate the data loaded into Snowflake.</li> 
</ol> 
<p>The solution is depicted in the following architecture diagram.<br /> <img alt="" class="size-full wp-image-62332 alignnone" height="512" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/04/10/BDB-4100-KDS-SNF-Arch.png" width="657" /></p> 
<h2>Prerequisites</h2> 
<p>You should have the following prerequisites:</p> 
<ul> 
 <li>An <a href="https://portal.aws.amazon.com/billing/signup/iam?nc2=h_ct&amp;redirect_url=https%3A%2F%2Faws.amazon.com%2Fregistration-confirmation&amp;src=header_signup#/support">AWS account</a> and access to the following AWS services: 
  <ul> 
   <li><a href="https://aws.amazon.com/iam/">AWS Identity and Access Management</a> (IAM)</li> 
   <li>Kinesis Data Streams</li> 
   <li>Amazon S3</li> 
   <li>Amazon Data Firehose</li> 
  </ul> </li> 
 <li>Familiarity with the <a href="http://aws.amazon.com/console">AWS Management Console</a>.</li> 
 <li>A <a href="https://signup.snowflake.com/">Snowflake account</a>.</li> 
 <li>A key pair generated and your user configured to connect securely to Snowflake. For instructions, refer to the following: 
  <ul> 
   <li><a href="https://docs.snowflake.com/en/user-guide/key-pair-auth#generate-the-private-key">Generate the private key</a></li> 
   <li><a href="https://docs.snowflake.com/en/user-guide/key-pair-auth#generate-a-public-key">Generate a public key</a></li> 
   <li><a href="https://docs.snowflake.com/en/user-guide/key-pair-auth#generate-a-public-key">Store the private and public keys securely</a></li> 
   <li><a href="https://docs.snowflake.com/en/user-guide/key-pair-auth#generate-a-public-key">Assign the public key to a Snowflake user</a></li> 
   <li><a href="https://docs.snowflake.com/en/user-guide/key-pair-auth#generate-a-public-key">Verify the user’s public key fingerprint</a></li> 
  </ul> </li> 
 <li>An S3 bucket for error logging.</li> 
 <li>The KDG set up. For instructions, refer to <a href="https://aws.amazon.com/blogs/big-data/test-your-streaming-data-solution-with-the-new-amazon-kinesis-data-generator/">Test Your Streaming Data Solution with the New Amazon Kinesis Data Generator</a>.</li> 
</ul> 
<h2><strong>Create a Snowflake database, schema, and table</strong></h2> 
<p>Complete the following steps to set up your data in Snowflake:</p> 
<ul> 
 <li>Log in to your Snowflake account and create the database: 
  <div class="hide-language"> 
   <pre><code class="lang-sql">create database adf_snf;</code></pre> 
  </div> </li> 
 <li>Create a schema in the new database: 
  <div class="hide-language"> 
   <pre><code class="lang-sql">create schema adf_snf.kds_blog;</code></pre> 
  </div> </li> 
 <li>Create a table in the new schema: 
  <div class="hide-language"> 
   <pre><code class="lang-sql">create or replace table iot_sensors
(sensorId number,
sensorType varchar,
internetIP varchar,
connectionTime timestamp_ntz,
currentTemperature number
);</code></pre> 
  </div> <p><img alt="" class="size-full wp-image-62333 alignnone" height="266" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/04/10/BDB-4100-SNF-1.png" style="margin: 10px 0px 10px 0px;" width="385" /></p></li> 
</ul> 
<h2>Create a Kinesis data stream</h2> 
<p>Complete the following steps to create your data stream:</p> 
<ul> 
 <li>On the Kinesis Data Streams console, choose <strong>Data streams</strong> in the navigation pane.</li> 
 <li>Choose <strong>Create data stream</strong>.<br /> <img alt="" class="aligncenter size-full wp-image-62335" height="163" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/04/10/BDB-4100-KDS-2.png" style="margin: 10px 0px 10px 0px;" width="1631" /></li> 
 <li>For Data stream name, enter a name (for example, <code>KDS-Demo-Stream</code>).</li> 
 <li>Leave the remaining settings as default.</li> 
 <li>Choose Create data stream.<br /> <img alt="" class="aligncenter size-full wp-image-62336" height="1200" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/04/10/BDB-4100-KDS-3.png" style="margin: 10px 0px 10px 0px;" width="1425" /></li> 
</ul> 
<h2>Create a Firehose delivery stream</h2> 
<p>Complete the following steps to create a Firehose delivery stream with Kinesis Data Streams as the source and Snowflake as its destination:</p> 
<ul> 
 <li>On the Amazon Data Firehose console, choose <strong>Create Firehose stream</strong>.</li> 
 <li>For <strong>Source</strong>, choose <strong>Amazon Kinesis Data Streams</strong>.</li> 
 <li>For <strong>Destination</strong>, choose <strong>Snowflake</strong>.</li> 
 <li>For <strong>Kinesis data stream</strong>, browse to the data stream you created earlier.</li> 
 <li>For <strong>Firehose stream name</strong>, leave the default generated name or enter a name of your preference.<br /> <img alt="" class="aligncenter size-full wp-image-62329" height="769" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/04/10/BDB-4100-ADF-2.png" style="margin: 10px 0px 10px 0px;" width="808" /></li> 
 <li>Under <strong>Connection settings</strong>, provide the following information to connect Amazon Data Firehose to Snowflake: 
  <ul> 
   <li>For <strong>Snowflake account URL</strong>, enter your Snowflake account URL.</li> 
   <li>For <strong>User</strong>, enter the user name generated in the prerequisites.</li> 
   <li>For <strong>Private key</strong>, enter the private key generated in the prerequisites. Make sure the private key is in PKCS8 format. Do not include the PEM <code>header-BEGIN</code> prefix and <code>footer-END</code> suffix as part of the private key. If the key is split across multiple lines, remove the line breaks.</li> 
   <li>For <strong>Role</strong>, select <strong>Use custom Snowflake role</strong> and enter the IAM role that has access to write to the database table.</li> 
  </ul> </li> 
</ul> 
<p>You can connect to Snowflake using public or private connectivity. If you don’t provide a VPC endpoint, the default connectivity mode is public. To allow list Firehose IPs in your Snowflake network policy, refer to <a href="https://docs.aws.amazon.com/firehose/latest/dev/create-destination.html#create-destination-snowflake">Choose Snowflake for Your Destination</a>. If you’re using a private link URL, provide the VPCE ID using <a href="https://docs.snowflake.com/en/sql-reference/functions/system_get_privatelink_config">SYSTEM$GET_PRIVATELINK_CONFIG</a>:</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">select SYSTEM$GET_PRIVATELINK_CONFIG();</code></pre> 
</div> 
<p>This function returns a JSON representation of the Snowflake account information necessary to facilitate the self-service configuration of private connectivity to the Snowflake service, as shown in the following screenshot.<br /> <img alt="" class="aligncenter size-full wp-image-62328" height="362" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/04/10/BDB-4100-KDS-SNF-pv1.png" style="margin: 10px 0px 10px 0px;" width="896" /></p> 
<ul> 
 <li>For this post, we’re using a private link, so for <strong>VPCE ID</strong>, enter the VPCE ID.<br /> <img alt="" class="aligncenter size-full wp-image-62327" height="1034" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/04/10/BDB-4100-ADF-3.png" style="margin: 10px 0px 10px 0px;" width="807" /></li> 
 <li>Under <strong>Database configuration settings</strong>, enter your Snowflake database, schema, and table names.</li> 
 <li>In the <strong>Backup settings</strong> section, for <strong>S3 backup bucket</strong>, enter the bucket you created as part of the prerequisites.</li> 
 <li>Choose <strong>Create Firehose stream</strong>.<br /> <img alt="" class="aligncenter size-full wp-image-62330" height="1125" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/04/10/BDB-4100-ADF-4.png" style="margin: 10px 0px 10px 0px;" width="815" /></li> 
</ul> 
<p>Alternatively, you can use an <a href="http://aws.amazon.com/cloudformation">AWS CloudFormation</a> template to create the Firehose delivery stream with Snowflake as the destination rather than using the Amazon Data Firehose console.</p> 
<p>To use the CloudFormation stack, choose</p> 
<p><a href="https://aws-blogs-artifacts-public.s3.amazonaws.com/BDB-4100/BDB-4100-ADF-SNF.yml" rel="noopener" target="_blank"><img alt="BDB-4100-CFN-Launch-Stack" class="alignnone wp-image-62392 size-full" height="27" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/04/11/cloudformation-launch-stack-BDB-4100.png" width="144" /></a></p> 
<p><strong>Generate sample stream data</strong><br /> Generate sample stream data from the KDG with the Kinesis data stream you created:</p> 
<div class="hide-language"> 
 <pre><code class="lang-json">{ 
"sensorId": {{random.number(999999999)}}, 
"sensorType": "{{random.arrayElement( ["Thermostat","SmartWaterHeater","HVACTemperatureSensor","WaterPurifier"] )}}", 
"internetIP": "{{internet.ip}}", 
"connectionTime": "{{date.now("YYYY-MM-DDTHH:m:ss")}}", 
"currentTemperature": {{random.number({"min":10,"max":150})}} 
}</code></pre> 
</div> 
<p><img alt="" class="aligncenter size-full wp-image-62331" height="672" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/04/10/BDB-4100-KDG-1.png" style="margin: 10px 0px 10px 0px;" width="959" /></p> 
<h3>Query the Snowflake table</h3> 
<p>Query the Snowflake table:</p> 
<div class="hide-language"> 
 <pre><code class="lang-sql">select * from adf_snf.kds_blog.iot_sensors;</code></pre> 
</div> 
<p>You can confirm that the data generated by the KDG that was sent to Kinesis Data Streams is loaded into the Snowflake table through Amazon Data Firehose.</p> 
<p><img alt="" class="aligncenter size-full wp-image-62334" height="309" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/04/10/BDB-4100-SNF-2.png" style="margin: 10px 0px 10px 0px;" width="858" /></p> 
<h2>Troubleshooting</h2> 
<p>If data is not loaded into Kinesis Data Steams after the KDG sends data to the Firehose delivery stream, refresh and make sure you are logged in to the KDG.</p> 
<p>If you made any changes to the Snowflake destination table definition, recreate the Firehose delivery stream.</p> 
<h2>Clean up</h2> 
<p>To avoid incurring future charges, delete the resources you created as part of this exercise if you are not planning to use them further.</p> 
<h2>Conclusion</h2> 
<p>Amazon Data Firehose provides a straightforward way to deliver data to Snowpipe Streaming, enabling you to save costs and reduce latency to seconds. To try Amazon Kinesis Firehose with Snowflake, refer to the Amazon Data Firehose with Snowflake as destination lab.</p> 
<hr /> 
<h3><strong>About the Authors</strong></h3> 
<p style="clear: both;"><img alt="" class="wp-image-59021 size-full alignleft" height="132" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/01/28/swapnab.jpg" width="100" /><strong>Swapna Bandla</strong>&nbsp;is a Senior Solutions Architect in the AWS Analytics Specialist SA Team. Swapna has a passion towards understanding customers data and analytics needs and empowering them to develop cloud-based well-architected solutions. Outside of work, she enjoys spending time with her family.</p> 
<p style="clear: both;"><img alt="" class="alignleft wp-image-62363 size-full" height="100" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/04/10/Mostafa-Mansour-100.jpg" width="100" /><strong>Mostafa Mansour</strong>&nbsp;is a Principal Product Manager – Tech at Amazon Web Services where he works on Amazon Kinesis Data Firehose. He specializes in developing intuitive product experiences that solve complex challenges for customers at scale. When he’s not hard at work on Amazon Kinesis Data Firehose, you’ll likely find Mostafa on the squash court, where he loves to take on challengers and perfect his dropshots.</p> 
<p style="clear: both;"><strong><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/25/bosco_profile.png"><img alt="" class="wp-image-55939 size-full alignleft" height="110" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/10/25/bosco_profile.png" width="100" /></a>Bosco Albuquerque</strong> is a Sr. Partner Solutions Architect at AWS and has over 20 years of experience working with database and analytics products from enterprise database vendors and cloud providers. He has helped technology companies design and implement data analytics solutions and products.</p>
