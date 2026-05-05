---
title: "Migrate from Amazon Kinesis Data Analytics for SQL Applications to Amazon Managed Service for Apache Flink Studio"
url: "https://aws.amazon.com/blogs/big-data/migrate-from-amazon-kinesis-data-analytics-for-sql-applications-to-amazon-managed-service-for-apache-flink-studio/"
date: "Thu, 29 Jun 2023 17:18:54 +0000"
author: "Nicholas Tunney"
feed_url: "https://aws.amazon.com/blogs/big-data/category/analytics/amazon-kinesis/kinesis-data-firehose/feed/"
---
<p><em><strong>February 9, 2024:</strong> Amazon Kinesis Data Firehose has been renamed to Amazon Data Firehose. Read the <a href="https://aws.amazon.com/about-aws/whats-new/2024/02/amazon-data-firehose-formerly-kinesis-data-firehose/" rel="noopener" target="_blank">AWS What’s New post</a> to learn more.</em></p> 
<p><em><strong>August 30, 2023: </strong>Amazon Kinesis Data Analytics has been renamed to Amazon Managed Service for Apache Flink. Read the announcement in the <a href="https://aws.amazon.com/blogs/aws/announcing-amazon-managed-service-for-apache-flink-renamed-from-amazon-kinesis-data-analytics" rel="noopener" target="_blank">AWS News Blog</a> and <a href="https://aws.amazon.com/managed-service-apache-flink/" rel="noopener" target="_blank">learn more</a>.</em></p> 
<p>In this post, we discuss why AWS recommends moving from Kinesis Data Analytics for SQL Applications to <a href="https://docs.aws.amazon.com/kinesisanalytics/latest/java/what-is.html" rel="noopener" target="_blank">Amazon Managed Service for Apache Flink</a> to take advantage of Apache Flink’s advanced streaming capabilities. We also show how to use <a href="https://docs.aws.amazon.com/kinesisanalytics/latest/java/what-is.html" rel="noopener" target="_blank">Amazon Managed Service for Apache Flink</a> Studio to test and tune your analysis before deploying your migrated applications. If you don’t have any Kinesis Data Analytics for SQL applications, this post still provides a background on many of the use cases you’ll see in your data analytics career and how <a href="https://docs.aws.amazon.com/kinesisanalytics/latest/java/what-is.html" rel="noopener" target="_blank">Amazon Managed Service for Apache Flink</a> can help you achieve your objectives.</p> 
<p><a href="https://docs.aws.amazon.com/kinesisanalytics/latest/java/what-is.html" rel="noopener" target="_blank">Amazon Managed Service for Apache Flink</a> is a fully managed Apache Flink service. You only need to upload your application JAR or executable, and AWS will manage the infrastructure and Flink job orchestration. To make things simpler, <a href="https://docs.aws.amazon.com/kinesisanalytics/latest/java/what-is.html" rel="noopener" target="_blank">Amazon Managed Service for Apache Flink</a> Studio is a notebook environment that uses Apache Flink and allows you to query data streams and develop SQL queries or proof of concept workloads before scaling your application to production in minutes.</p> 
<p>We recommend that you use <a href="https://docs.aws.amazon.com/kinesisanalytics/latest/java/what-is.html" rel="noopener" target="_blank">Amazon Managed Service for Apache Flink</a> or <a href="https://docs.aws.amazon.com/kinesisanalytics/latest/java/what-is.html" rel="noopener" target="_blank">Amazon Managed Service for Apache Flink</a> Studio over Kinesis Data Analytics for SQL. This is because <a href="https://docs.aws.amazon.com/kinesisanalytics/latest/java/what-is.html" rel="noopener" target="_blank">Amazon Managed Service for Apache Flink</a> and <a href="https://docs.aws.amazon.com/kinesisanalytics/latest/java/what-is.html" rel="noopener" target="_blank">Amazon Managed Service for Apache Flink</a> Studio offer advanced data stream processing features, including exactly-once processing semantics, event time windows, extensibility using user-defined functions (UDFs) and custom integrations, imperative language support, durable application state, horizontal scaling, support for multiple data sources, and more. These are critical for ensuring accuracy, completeness, consistency, and reliability of data stream processing and are not available with Kinesis Data Analytics for SQL.</p> 
<h2>Solution overview</h2> 
<p>For our use case, we use several AWS services to stream, ingest, transform, and analyze sample automotive sensor data in real time using <a href="https://docs.aws.amazon.com/kinesisanalytics/latest/java/what-is.html" rel="noopener" target="_blank">Amazon Managed Service for Apache Flink</a> Studio. <a href="https://docs.aws.amazon.com/kinesisanalytics/latest/java/what-is.html" rel="noopener" target="_blank">Amazon Managed Service for Apache Flink</a> Studio allows us to create a notebook, which is a web-based development environment. With notebooks, you get a simple interactive development experience combined with the advanced capabilities provided by Apache Flink. <a href="https://docs.aws.amazon.com/kinesisanalytics/latest/java/what-is.html" rel="noopener" target="_blank">Amazon Managed Service for Apache Flink</a> Studio uses <a href="https://zeppelin.apache.org/" rel="noopener" target="_blank">Apache Zeppelin</a> as the notebook, and uses <a href="https://flink.apache.org/" rel="noopener" target="_blank">Apache Flink</a> as the stream processing engine. <a href="https://docs.aws.amazon.com/kinesisanalytics/latest/java/what-is.html" rel="noopener" target="_blank">Amazon Managed Service for Apache Flink</a> Studio notebooks seamlessly combine these technologies to make advanced analytics on data streams accessible to developers of all skill sets. Notebooks are provisioned quickly and provide a way for you to instantly view and analyze your streaming data. Apache Zeppelin provides your Studio notebooks with a complete suite of analytics tools, including the following:</p> 
<ul> 
 <li>Data visualization</li> 
 <li>Exporting data to files</li> 
 <li>Controlling the output format for easier analysis</li> 
 <li>Ability to turn the notebook into a scalable, production application</li> 
</ul> 
<p>Unlike Kinesis Data Analytics for SQL Applications, <a href="https://docs.aws.amazon.com/kinesisanalytics/latest/java/what-is.html" rel="noopener" target="_blank">Amazon Managed Service for Apache Flink</a> adds the <a href="https://docs.aws.amazon.com/kinesisanalytics/latest/java/how-zeppelin-kdasql.html" rel="noopener" target="_blank">following SQL support</a>:</p> 
<ul> 
 <li>Joining stream data between multiple Kinesis data streams, or between a Kinesis data stream and an <a href="https://aws.amazon.com/msk/" rel="noopener" target="_blank">Amazon Managed Streaming for Apache Kafka</a> (Amazon MSK) topic</li> 
 <li>Real-time visualization of transformed data in a data stream</li> 
 <li>Using Python scripts or Scala programs within the same application</li> 
 <li>Changing offsets of the streaming layer</li> 
</ul> 
<p>Another benefit of <a href="https://docs.aws.amazon.com/kinesisanalytics/latest/java/what-is.html" rel="noopener" target="_blank">Amazon Managed Service for Apache Flink</a> is the improved scalability of the solution once deployed, because you can scale the underlying resources to meet demand. In Kinesis Data Analytics for SQL Applications, scaling is performed by adding more pumps to persuade the application into adding more resources.</p> 
<p>In our solution, we create a notebook to access automotive sensor data, enrich the data, and send the enriched output from the Amazon Managed Service for Apache Flink Studio notebook to an <a href="https://aws.amazon.com/kinesis/data-firehose/" rel="noopener" target="_blank">Amazon Kinesis Data Firehose</a> delivery stream for delivery to an <a href="http://aws.amazon.com/s3" rel="noopener" target="_blank">Amazon Simple Storage Service</a> (Amazon S3) data lake. This pipeline could further be used to send data to <a href="https://aws.amazon.com/opensearch-service/" rel="noopener" target="_blank">Amazon OpenSearch Service</a> or other targets for additional processing and visualization.</p> 
<h2>Kinesis Data Analytics for SQL Applications vs. <strong>Amazon Managed Service for Apache Flink</strong></h2> 
<p>In our example, we perform the following actions on the streaming data:</p> 
<ol> 
 <li>Connect to an <a href="https://aws.amazon.com/kinesis/data-streams/" rel="noopener" target="_blank">Amazon Kinesis Data Streams</a> data stream.</li> 
 <li>View the stream data.</li> 
 <li>Transform and enrich the data.</li> 
 <li>Manipulate the data with Python.</li> 
 <li>Restream the data to a Firehose delivery stream.</li> 
</ol> 
<p>To compare Kinesis Data Analytics for SQL Applications with <a href="https://docs.aws.amazon.com/kinesisanalytics/latest/java/what-is.html" rel="noopener" target="_blank">Amazon Managed Service for Apache Flink</a>, let’s first discuss how Kinesis Data Analytics for SQL Applications works.</p> 
<p>At the root of a Kinesis Data Analytics for SQL application is the concept of an in-application stream. You can think of the in-application stream as a table that holds the streaming data so you can perform actions on it. The in-application stream is mapped to a streaming source such as a Kinesis data stream. To get data into the in-application stream, first set up a source in the management console for your Kinesis Data Analytics for SQL application. Then, create a pump that reads data from the source stream and places it into the table. The pump query runs continuously and feeds the source data into the in-application stream. You can create multiple pumps from multiple sources to feed the in-application stream. Queries are then run on the in-application stream, and results can be interpreted or sent to other destinations for further processing or storage.</p> 
<p>The following SQL demonstrates setting up an in-application stream and pump:</p> 
<pre><code class="lang-sql">CREATE OR REPLACE STREAM "TEMPSTREAM" ( 
   "column1" BIGINT NOT NULL, 
   "column2" INTEGER, 
   "column3" VARCHAR(64));

CREATE OR REPLACE PUMP "SAMPLEPUMP" AS 
INSERT INTO "TEMPSTREAM" ("column1", 
                          "column2", 
                          "column3") 
SELECT STREAM inputcolumn1, 
      inputcolumn2, 
      inputcolumn3
FROM "INPUTSTREAM";</code></pre> 
<p>Data can be read from the in-application stream using a SQL SELECT query:</p> 
<pre><code class="lang-sql">SELECT *
FROM "TEMPSTREAM"</code></pre> 
<p>When creating the same setup in Amazon Managed Service for Apache Flink Studio, you use the underlying Apache Flink environment to connect to the streaming source, and create the data stream in one statement using a connector. The following example shows connecting to the same source we used before, but using Apache Flink:</p> 
<pre><code class="lang-sql">CREATE TABLE `MY_TABLE` ( 
   "column1" BIGINT NOT NULL, 
   "column2" INTEGER, 
   "column3" VARCHAR(64)
) WITH (
   'connector' = 'kinesis',
   'stream' = sample-kinesis-stream',
   'aws.region' = 'aws-kinesis-region',
   'scan.stream.initpos' = 'LATEST',
   'format' = 'json'
 );</code></pre> 
<p><code>MY_TABLE</code> is now a data stream that will continually receive the data from our sample Kinesis data stream. It can be queried using a SQL SELECT statement:</p> 
<pre><code class="lang-sql">SELECT column1, 
       column2, 
       column3
FROM MY_TABLE;</code></pre> 
<p>Although Kinesis Data Analytics for SQL Applications use a <a href="https://docs.aws.amazon.com/kinesisanalytics/latest/sqlref/analytics-sql-reference.html" rel="noopener" target="_blank">subset of the SQL:2008 standard</a> with extensions to enable operations on streaming data, <a href="https://nightlies.apache.org/flink/flink-docs-master/docs/dev/table/sql/overview/" rel="noopener" target="_blank">Apache Flink’s SQL support</a> is based on <a href="https://calcite.apache.org/" rel="noopener" target="_blank">Apache Calcite</a>, which implements the SQL standard.</p> 
<p>It’s also important to mention that Amazon Managed Service for Apache Flink Studio supports PyFlink and Scala alongside SQL within the same notebook. This allows you to perform complex, programmatic methods on your streaming data that aren’t possible with SQL.</p> 
<h2>Prerequisites</h2> 
<p>During this exercise, we set up various AWS resources and perform analytics queries. To follow along, you need an AWS account with administrator access. If you don’t already have an AWS account with administrator access, <a href="https://aws.amazon.com/getting-started/" rel="noopener" target="_blank">create one now</a>. The services outlined in this post may incur charges to your AWS account. Make sure to follow the cleanup instructions at the end of this post.</p> 
<h2>Configure streaming data</h2> 
<p>In the streaming domain, we’re often tasked with exploring, transforming, and enriching data coming from Internet of Things (IoT) sensors. To generate the real-time sensor data, we employ the <a href="https://docs.aws.amazon.com/solutions/latest/iot-device-simulator/welcome.html" rel="noopener" target="_blank">AWS IoT Device Simulator</a>. This simulator runs within your AWS account and provides a web interface that lets users launch fleets of virtually connected devices from a user-defined template and then simulate them to publish data at regular intervals to <a href="https://aws.amazon.com/iot-core/" rel="noopener" target="_blank">AWS IoT Core</a>. This means we can build a virtual fleet of devices to generate sample data for this exercise.</p> 
<p>We deploy the IoT Device Simulator using the following <a href="https://aws.amazon.com/cloudfront/" rel="noopener" target="_blank">Amazon CloudFront</a> <a href="https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?templateURL=https:%2F%2Fs3.amazonaws.com%2Fsolutions-reference%2Fiot-device-simulator%2Flatest%2Fiot-device-simulator.template" rel="noopener" target="_blank">template</a>. It handles creating all the necessary resources in your account.</p> 
<ol> 
 <li>On the <strong>Specify stack details</strong> page, assign a name to your solution stack.</li> 
 <li>Under <strong>Parameters</strong>, review the parameters for this solution template and modify them as necessary.</li> 
 <li>For <strong>User email</strong>, enter a valid email to receive a link and password to log in to the IoT Device Simulator UI.</li> 
 <li>Choose <strong>Next</strong>.</li> 
 <li>On the <strong>Configure stack options</strong> page, choose <strong>Next</strong>.</li> 
 <li>On the <strong>Review</strong> page, review and confirm the settings. Select the check boxes acknowledging that the template creates <a href="http://aws.amazon.com/iam" rel="noopener" target="_blank">AWS Identity and Access Management</a> (IAM) resources.</li> 
 <li>Choose <strong>Create stack</strong>.</li> 
</ol> 
<p>The stack takes about 10 minutes to install.</p> 
<ol start="8"> 
 <li>When you receive your invitation email, choose the CloudFront link and log in to the IoT Device Simulator using the credentials provided in the email.</li> 
</ol> 
<p>The solution contains a prebuilt automotive demo that we can use to begin delivering sensor data quickly to AWS.</p> 
<ol start="9"> 
 <li>On the <strong>Device Type</strong> page, choose <strong>Create Device Type</strong>.</li> 
 <li>Choose <strong>Automotive Demo</strong>.</li> 
 <li>The payload is auto populated. Enter a name for your device, and enter <code>automotive-topic</code> as the topic.<br /> <img alt="" class="aligncenter wp-image-38560 size-full" height="140" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/30/image002-Medium.png" style="margin: 10px 0px 10px 0px;" width="640" /></li> 
 <li>Choose <strong>Save</strong>.</li> 
</ol> 
<p>Now we create a simulation.</p> 
<ol start="13"> 
 <li>On the <strong>Simulations</strong> page, choose <strong>Create Simulation</strong>.</li> 
 <li>For <strong>Simulation type</strong>, choose<strong> Automotive Demo</strong>.</li> 
 <li>For <strong>Select a device type</strong>, choose the demo device you created.</li> 
 <li>For <strong>Data transmission interval</strong> and <strong>Data transmission duration</strong>, enter your desired values.</li> 
</ol> 
<p>You can enter any values you like, but use at least 10 devices transmitting every 10 seconds. You’ll want to set your data transmission duration to a few minutes, or you’ll need to restart your simulation several times during the lab.</p> 
<ol start="17"> 
 <li>Choose<strong> Save</strong>.<br /> <img alt="" class="alignnone wp-image-38991 size-full" height="307" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/30/image003-Medium.png" style="margin: 10px 0px 10px 0px;" width="640" /></li> 
</ol> 
<p>Now we can run the simulation.</p> 
<ol start="18"> 
 <li>On the <strong>Simulations</strong> page, select the desired simulation, and choose <strong>Start simulations</strong>.</li> 
</ol> 
<p>Alternatively, choose <strong>View</strong> next to the simulation you want to run, then choose <strong>Start</strong> to run the simulation.</p> 
<ol start="19"> 
 <li>To view the simulation, choose <strong>View</strong> next to the simulation you want to view.</li> 
</ol> 
<p>If the simulation is running, you can view a map with the locations of the devices, and up to 100 of the most recent messages sent to the IoT topic.</p> 
<p>We can now check to ensure our simulator is sending the sensor data to AWS IoT Core.</p> 
<ol start="20"> 
 <li>Navigate to the AWS IoT Core console.</li> 
</ol> 
<p>Make sure you’re in the same Region you deployed your IoT Device Simulator.</p> 
<ol start="21"> 
 <li>In the navigation pane, choose <strong>MQTT Test Client</strong>.</li> 
 <li>Enter the topic filter <code>automotive-topic</code> and choose <strong>Subscribe</strong>.</li> 
</ol> 
<p>As long as you have your simulation running, the messages being sent to the IoT topic will be displayed.</p> 
<p><img alt="" class="alignnone wp-image-38562 size-full" height="119" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/30/image004-Medium.png" style="margin: 10px 0px 10px 0px;" width="640" /></p> 
<p>Finally, we can set a rule to route the IoT messages to a Kinesis data stream. This stream will provide our source data for the Amazon Managed Service for Apache Flink Studio notebook.</p> 
<ol start="23"> 
 <li>On the AWS IoT Core console, choose <strong>Message Routing</strong> and <strong>Rules</strong>.</li> 
 <li>Enter a name for the rule, such as <code>automotive_route_kinesis</code>, then choose <strong>Next</strong>.<br /> <img alt="" class="alignnone wp-image-38993 size-full" height="296" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/30/image005-Medium.png" style="margin: 10px 0px 10px 0px;" width="640" /></li> 
 <li>Provide the following SQL statement. This SQL will select all message columns from the <code>automotive-topic</code> the IoT Device Simulator is publishing:</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-sql">SELECT timestamp, trip_id, VIN, brake, steeringWheelAngle, torqueAtTransmission, engineSpeed, vehicleSpeed, acceleration, parkingBrakeStatus, brakePedalStatus, transmissionGearPosition, gearLeverPosition, odometer, ignitionStatus, fuelLevel, fuelConsumedSinceRestart, oilTemp, location 
FROM 'automotive-topic' WHERE 1=1</code></pre> 
</div> 
<ol start="26"> 
 <li>Choose <strong>Next</strong>.</li> 
 <li>Under <strong>Rule Actions</strong>, select <strong>Kinesis Stream</strong> as the source.</li> 
 <li>Choose <strong>Create New Kinesis Stream</strong>.</li> 
</ol> 
<p>This opens a new window.</p> 
<ol start="29"> 
 <li>For <strong>Data stream name</strong>, enter <code>automotive-data</code>.</li> 
</ol> 
<p>We use a provisioned stream for this exercise.</p> 
<p><img alt="" class="alignnone wp-image-38994 size-full" height="150" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/30/image006-Medium.png" style="margin: 10px 0px 10px 0px;" width="640" /></p> 
<ol start="30"> 
 <li>Choose <strong>Create Data Stream</strong>.</li> 
</ol> 
<p>You may now close this window and return to the AWS IoT Core console.</p> 
<ol start="31"> 
 <li>Choose the refresh button next to <strong>Stream name</strong>, and choose the <code>automotive-data</code> stream.<br /> <img alt="" class="alignnone wp-image-38995 size-full" height="344" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/30/image007-Medium.png" style="margin: 10px 0px 10px 0px;" width="640" /></li> 
 <li>Choose <strong>Create new role</strong> and name the role <code>automotive-role</code>.</li> 
 <li>Choose <strong>Next</strong>.</li> 
 <li>Review the rule properties, and choose <strong>Create</strong>.</li> 
</ol> 
<p>The rule begins routing data immediately.</p> 
<h2>Set up <strong>Amazon Managed Service for Apache Flink </strong> Studio</h2> 
<p>Now that we have our data streaming through AWS IoT Core and into a Kinesis data stream, we can create our Amazon Managed Service for Apache Flink Studio notebook.</p> 
<ol> 
 <li>On the Amazon Kinesis console, choose <strong>Analytics applications</strong> in the navigation pane.</li> 
 <li>On the <strong>Studio </strong>tab, choose <strong>Create Studio notebook</strong>.<br /> <img alt="" class="alignnone wp-image-38996 size-full" height="187" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/30/image008-Medium.png" style="margin: 10px 0px 10px 0px;" width="640" /></li> 
 <li>Leave <strong>Quick create with sample code</strong> selected.</li> 
 <li>Name the notebook <code>automotive-data-notebook</code>.<br /> <img alt="" class="alignnone wp-image-38997 size-full" height="531" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/30/image009-Medium.png" style="margin: 10px 0px 10px 0px;" width="640" /></li> 
 <li>Choose <strong>Create</strong> to create a new <a href="https://aws.amazon.com/glue" rel="noopener" target="_blank">AWS Glue</a> database in a new window.</li> 
 <li>Choose <strong>Add database</strong>.<br /> <img alt="" class="alignnone wp-image-38998 size-full" height="113" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/30/image010-Medium.png" style="margin: 10px 0px 10px 0px;" width="640" /></li> 
 <li>Name the database <code>automotive-notebook-glue</code>.</li> 
 <li>Choose <strong>Create</strong>.<br /> <img alt="" class="alignnone wp-image-38999 size-full" height="424" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/30/image011-Medium.png" style="margin: 10px 0px 10px 0px;" width="602" /></li> 
 <li>Return to the <strong>Create Studio notebook</strong> section.</li> 
 <li>Choose refresh and choose your new AWS Glue database.</li> 
 <li>Choose <strong>Create Studio notebook</strong>.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/12/01/permissions-image012-7-300x262-1.png" rel="noopener" target="_blank"><img alt="" class="alignnone wp-image-39148 size-full" height="262" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/12/01/permissions-image012-7-300x262-1.png" style="margin: 10px 0px 10px 0px;" width="300" /></a></li> 
 <li>To start the Studio notebook, choose <strong>Run</strong> and confirm.<br /> <img alt="" class="alignnone wp-image-40858 size-full" height="239" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/01/03/BDB-2461-image013-Medium.png" style="margin: 10px 0px 10px 0px;" width="640" /></li> 
 <li>Once the notebook is running, choose the notebook and choose <strong>Open in Apache Zeppelin</strong>.</li> 
 <li>Choose <strong>Import note</strong>.</li> 
 <li>Choose <strong>Add from URL</strong>.<br /> <a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/12/01/cloudtrail-image014-7-300x251-1.png" rel="noopener" target="_blank"><img alt="" class="alignnone wp-image-39149 size-full" height="251" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/12/01/cloudtrail-image014-7-300x251-1.png" style="margin: 10px 0px 10px 0px;" width="300" /></a></li> 
 <li>Enter the following URL: <code>https://aws-blogs-artifacts-public.s3.amazonaws.com/artifacts/BDB-2461/auto-notebook.ipynb</code>.</li> 
 <li>Choose <strong>Import Note</strong>.<br /> <img alt="" class="alignnone wp-image-38573 size-full" height="551" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/28/image015-4.png" style="margin: 10px 0px 10px 0px;" width="601" /></li> 
 <li>Open the new note.</li> 
</ol> 
<h2>Perform stream analysis</h2> 
<p>In a Kinesis Data Analytics for SQL application, we add our streaming course via the management console, and then define an in-application stream and pump to stream data from our Kinesis data stream. The in-application stream functions as a table to hold the data and make it available for us to query. The pump takes the data from our source and streams it to our in-application stream. Queries may then be run against the in-application stream using SQL, just as we’d query any SQL table. See the following code:</p> 
<pre><code class="lang-sql">CREATE OR REPLACE STREAM "AUTOSTREAM" ( 
    `trip_id` CHAR(36),
    `VIN` CHAR(17),
    `brake` FLOAT,
    `steeringWheelAngle` FLOAT,
    `torqueAtTransmission` FLOAT,
    `engineSpeed` FLOAT,
    `vehicleSpeed` FLOAT,
    `acceleration` FLOAT,
    `parkingBrakeStatus` BOOLEAN,
    `brakePedalStatus` BOOLEAN,
    `transmissionGearPosition` VARCHAR(10),
    `gearLeverPosition` VARCHAR(10),
    `odometer` FLOAT,
    `ignitionStatus` VARCHAR(4),
    `fuelLevel` FLOAT,
    `fuelConsumedSinceRestart` FLOAT,
    `oilTemp` FLOAT,
    `location` VARCHAR(100),
    `timestamp` TIMESTAMP(3));

CREATE OR REPLACE PUMP "MYPUMP" AS 
INSERT INTO "AUTOSTREAM" ("trip_id",
    "VIN",
    "brake",
    "steeringWheelAngle",
    "torqueAtTransmission",
    "engineSpeed",
    "vehicleSpeed",
    "acceleration",
    "parkingBrakeStatus",
    "brakePedalStatus",
    "transmissionGearPosition",
    "gearLeverPosition",
    "odometer",
    "ignitionStatus",
    "fuelLevel",
    "fuelConsumedSinceRestart",
    "oilTemp",
    "location",
    "timestamp")
SELECT VIN,
    brake,
    steeringWheelAngle,
    torqueAtTransmission,
    engineSpeed,
    vehicleSpeed,
    acceleration,
    parkingBrakeStatus,
    brakePedalStatus,
    transmissionGearPosition,
    gearLeverPosition,
    odometer,
    ignitionStatus,
    fuelLevel,
    fuelConsumedSinceRestart,
    oilTemp,
    location,
    timestamp
FROM "INPUT_STREAM"
</code></pre> 
<p>To migrate an in-application stream and pump from our Kinesis Data Analytics for SQL application to Amazon Managed Service for Apache Flink Studio, we convert this into a single CREATE statement by removing the pump definition and defining a <code>kinesis</code> connector. The first paragraph in the Zeppelin notebook sets up a connector that is presented as a table. We can define columns for all items in the incoming message, or a subset.</p> 
<p><img alt="" class="alignnone wp-image-40762 size-full" height="384" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/12/29/para1-Medium.png" style="margin: 10px 0px 10px 0px;" width="640" /></p> 
<p>Run the statement, and a success result is output in your notebook. We can now query this table using SQL, or we can perform programmatic operations with this data using PyFlink or Scala.</p> 
<p>Before performing real-time analytics on the streaming data, let’s look at how the data is currently formatted. To do this, we run a simple Flink SQL query on the table we just created. The SQL used in our streaming application is identical to what is used in a SQL application.</p> 
<p><img alt="" class="alignnone wp-image-40763 size-full" height="61" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/12/29/para2-Medium.png" style="margin: 10px 0px 10px 0px;" width="640" /></p> 
<p>Note that if you don’t see records after a few seconds, make sure that your IoT Device Simulator is still running.</p> 
<p>If you’re also running the Kinesis Data Analytics for SQL code, you may see a slightly different result set. This is another key differentiator in Amazon Managed Service for Apache Flink, because the latter has the concept of exactly once delivery. If this application is deployed to production and is restarted or if scaling actions occur, Amazon Managed Service for Apache Flink ensures you only receive each message once, whereas in a Kinesis Data Analytics for SQL application, you need to further process the incoming stream to ensure you ignore repeat messages that could affect your results.</p> 
<p>You can stop the current paragraph by choosing the pause icon. You may see an error displayed in your notebook when you stop the query, but it can be ignored. It’s just letting you know that the process was canceled.</p> 
<p><img alt="" class="alignnone wp-image-38576 size-full" height="160" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/28/image018-6.png" style="margin: 10px 0px 10px 0px;" width="270" /></p> 
<p>Flink SQL implements the SQL standard, and provides an easy way to perform calculations on the stream data just like you would when querying a database table. A common task while enriching data is to create a new field to store a calculation or conversion (such as from Fahrenheit to Celsius), or create new data to provide simpler queries or improved visualizations downstream. Run the next paragraph to see how we can add a Boolean value named <code>accelerating</code>, which we can easily use in our sink to know if an automobile was currently accelerating at the time the sensor was read. The process here doesn’t differ between Kinesis Data Analytics for SQL and Amazon Managed Service for Apache Flink.</p> 
<p><img alt="" class="alignnone wp-image-40764 size-full" height="140" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/12/29/para3-Medium.png" style="margin: 10px 0px 10px 0px;" width="640" /></p> 
<p>You can stop the paragraph from running when you have inspected the new column, comparing our new Boolean value to the FLOAT <code>acceleration</code> column.</p> 
<p>Data being sent from a sensor is usually compact to improve latency and performance. Being able to enrich the data stream with external data and enrich the stream, such as additional vehicle information or current weather data, can be very useful. In this example, let’s assume we want to bring in data currently stored in a CSV in Amazon S3, and add a column named color that reflects the current engine speed band.</p> 
<p>Apache Flink SQL provides several <a href="https://docs.ververica.com/user_guide/sql_development/connectors.html#id1" rel="noopener" target="_blank">source connectors</a> for AWS services and other sources. Creating a new table like we did in our first paragraph but instead using the filesystem connector permits Flink to directly connect to Amazon S3 and read our source data. Previously in Kinesis Data Analytics for SQL Applications, you couldn’t add new references inline. Instead, you <a href="https://docs.aws.amazon.com/kinesisanalytics/latest/dev/app-add-reference-data.html" rel="noopener" target="_blank">defined S3 reference data</a> and added it to your application configuration, which you could then use as a reference in a SQL JOIN.</p> 
<p>NOTE: If you are not using the us-east-1 region, you can <a href="https://aws-blogs-artifacts-public.s3.amazonaws.com/artifacts/BDB-2461/colors.csv" rel="noopener" target="_blank">download the csv</a> and place the object your own S3 bucket.&nbsp; Reference the csv file as <code>s3a://&lt;bucket-name&gt;/&lt;key-name&gt;</code></p> 
<p><img alt="" class="alignnone wp-image-40765 size-full" height="172" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/12/29/para4-Medium.png" style="margin: 10px 0px 10px 0px;" width="640" /></p> 
<p>Building on the last query, the next paragraph performs a SQL JOIN on our current data and the new lookup source table we created.</p> 
<p><img alt="" class="alignnone wp-image-40766 size-full" height="176" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/12/29/para5-Medium.png" style="margin: 10px 0px 10px 0px;" width="640" /></p> 
<p>Now that we have an enriched data stream, we restream this data. In a real-world scenario, we have many choices on what to do with our data, such as sending the data to an S3 data lake, another Kinesis data stream for further analysis, or storing the data in OpenSearch Service for visualization. For simplicity, we send the data to Kinesis Data Firehose, which streams the data into an S3 bucket acting as our data lake.</p> 
<p>Kinesis Data Firehose can stream data to Amazon S3, OpenSearch Service, <a href="http://aws.amazon.com/redshift" rel="noopener" target="_blank">Amazon Redshift</a> data warehouses, and Splunk in just a few clicks.</p> 
<h2>Create the Kinesis Data Firehose delivery stream</h2> 
<p>To create our delivery stream, complete the following steps:</p> 
<ol> 
 <li>On the Kinesis Data Firehose console, choose <strong>Create delivery stream</strong>.</li> 
 <li>Choose <strong>Direct PUT</strong> for the stream source and <strong>Amazon S3</strong> as the target.<br /> <img alt="" class="alignnone wp-image-38580 size-full" height="283" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/28/image022-5.png" style="margin: 10px 0px 10px 0px;" width="812" /></li> 
 <li>Name your delivery stream automotive-firehose.<br /> <img alt="" class="alignnone wp-image-38581 size-full" height="184" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/28/image023-4.png" style="margin: 10px 0px 10px 0px;" width="806" /></li> 
 <li>Under <strong>Destination settings</strong>, create a new bucket or use an existing bucket.</li> 
 <li>Take note of the S3 bucket URL.<br /> <img alt="" class="alignnone wp-image-39012 size-full" height="280" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/30/image024-Medium.png" style="margin: 10px 0px 10px 0px;" width="640" /></li> 
 <li>Choose <strong>Create delivery stream</strong>.</li> 
</ol> 
<p>The stream takes a few seconds to create.</p> 
<ol start="7"> 
 <li>Return to the Amazon Managed Service for Apache Flink console and choose <strong>Streaming applications</strong>.</li> 
 <li>On the <strong>Studio</strong> tab, and choose your Studio notebook.<br /> <img alt="" class="alignnone wp-image-39013 size-full" height="142" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/30/image025-Medium.png" style="margin: 10px 0px 10px 0px;" width="640" /></li> 
 <li>Choose the link under <strong>IAM role</strong>.<br /> <img alt="" class="alignnone wp-image-40859 size-full" height="271" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/01/03/BDB-2461-image026-Medium.png" style="margin: 10px 0px 10px 0px;" width="640" /></li> 
 <li>In the IAM window, choose <strong>Add permissions</strong> and <strong>Attach policies</strong>.<br /> <img alt="" class="alignnone wp-image-40860 size-full" height="251" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/01/03/BDB-2461-image027-Medium.png" style="margin: 10px 0px 10px 0px;" width="640" /></li> 
 <li>Search for and select AmazonKinesisFullAccess and CloudWatchFullAccess, then choose <strong>Attach policy</strong>.</li> 
 <li>You may return to your Zeppelin notebook.</li> 
</ol> 
<h2>Stream data into Kinesis Data Firehose</h2> 
<p>As of Apache Flink v1.15, creating the connector to the Firehose delivery stream works similar to creating a connector to any Kinesis data stream. Note that there are two differences: the connector is <code>firehose</code>, and the stream attribute becomes <code>delivery-stream</code>.</p> 
<p><img alt="" class="alignnone wp-image-40767 size-full" height="365" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/12/29/para6-Medium.png" style="margin: 10px 0px 10px 0px;" width="640" /></p> 
<p>After the connector is created, we can write to the connector like any SQL table.</p> 
<p><img alt="" class="alignnone wp-image-40768 size-full" height="174" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/12/29/para7-Medium.png" style="margin: 10px 0px 10px 0px;" width="640" /></p> 
<p>To validate that we’re getting data through the delivery stream, open the Amazon S3 console and confirm you see files being created. Open the file to inspect the new data.</p> 
<p>In Kinesis Data Analytics for SQL Applications, we would have created a new destination in the SQL application dashboard. To migrate an existing destination, you add a SQL statement to your notebook that defines the new destination right in the code. You can continue to write to the new destination as you would have with an INSERT while referencing the new table name.</p> 
<h2>Time data</h2> 
<p>Another common operation you can perform in Amazon Managed Service for Apache Flink Studio notebooks is aggregation over a window of time. This sort of data can be used to send to another Kinesis data stream to identify anomalies, send alerts, or be stored for further processing. The next paragraph contains a SQL query that uses a tumbling window and aggregates total fuel consumed for the automotive fleet for 30-second periods. Like our last example, we could connect to another data stream and insert this data for further analysis.</p> 
<p><img alt="" class="alignnone wp-image-40769 size-full" height="84" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/12/29/para8-Medium.png" style="margin: 10px 0px 10px 0px;" width="640" /></p> 
<h2>Scala and PyFlink</h2> 
<p>There are times when a function you’d perform on your data stream is better written in a programming language instead of SQL, for both simplicity and maintenance. Some examples include complex calculations that SQL functions don’t support natively, certain string manipulations, the splitting of data into multiple streams, and interacting with other AWS services (such as text translation or sentiment analysis). Amazon Managed Service for Apache Flink has the ability to use multiple <a href="https://docs.aws.amazon.com/kinesisanalytics/latest/java/how-zeppelin-interactive.html#how-zeppelin-interactive-interpreters" rel="noopener" target="_blank">Flink interpreters</a> within the Zeppelin notebook, which is not available in Kinesis Data Analytics for SQL Applications.</p> 
<p>If you have been paying close attention to our data, you’ll see that the location field is a JSON string. In Kinesis Data Analytics for SQL, we could use string functions and define a <a href="https://docs.aws.amazon.com/kinesisanalytics/latest/sqlref/sql-reference-create-function.html" rel="noopener" target="_blank">SQL function</a> and break apart the JSON string. This is a fragile approach depending on the stability of the message data, but this could be improved with several SQL functions. The syntax for creating a function in Kinesis Data Analytics for SQL follows this pattern:</p> 
<pre><code class="lang-sql">CREATE FUNCTION ''&lt;function_name&gt;'' ( ''&lt;parameter_list&gt;'' )
    RETURNS ''&lt;data type&gt;''
    LANGUAGE SQL
    [ SPECIFIC ''&lt;specific_function_name&gt;''  | [NOT] DETERMINISTIC ]
    CONTAINS SQL
    [ READS SQL DATA ]
    [ MODIFIES SQL DATA ]
    [ RETURNS NULL ON NULL INPUT | CALLED ON NULL INPUT ]  
  RETURN ''&lt;SQL-defined function body&gt;''
</code></pre> 
<p>In Amazon Managed Service for Apache Flink, AWS recently upgraded the Apache Flink environment to v1.15, which extends Apache Flink SQL’s table SQL to <a href="https://nightlies.apache.org/flink/flink-docs-release-1.15/docs/dev/table/functions/systemfunctions/#json-functions" rel="noopener" target="_blank">add JSON functions</a> that are similar to JSON Path syntax. This allows us to query the JSON string directly in our SQL. See the following code:</p> 
<pre><code class="lang-python">%flink.ssql(type=update)
SELECT JSON_STRING(location, ‘$.latitude) AS latitude,
JSON_STRING(location, ‘$.longitude) AS longitude
FROM my_table</code></pre> 
<p>Alternatively, and required prior to Apache Flink v1.15, we can use Scala or PyFlink in our notebook to convert the field and restream the data. Both languages provide robust JSON string handling.</p> 
<p>The following PyFlink code defines two <a href="https://flink.apache.org/2020/04/09/pyflink-udf-support-flink.html" rel="noopener" target="_blank">user-defined functions</a>, which extract the latitude and longitude from the location field of our message. These UDFs can then be invoked from using Flink SQL. We reference the environment variable st_env. PyFlink creates <a href="https://zeppelin.apache.org/docs/0.9.0/interpreter/flink.html#pyflinkflinkpyflink" rel="noopener" target="_blank">six variables</a> for you in your Zeppelin notebook. Zeppelin also exposes a <a href="https://zeppelin.apache.org/docs/0.9.0/usage/other_features/zeppelin_context.html" rel="noopener" target="_blank">context</a> for you as the variable z.</p> 
<p><img alt="" class="alignnone wp-image-40770 size-full" height="264" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/12/29/para9-Medium.png" style="margin: 10px 0px 10px 0px;" width="640" /></p> 
<p><img alt="" class="alignnone wp-image-40771 size-full" height="64" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/12/29/para10-Medium.png" style="margin: 10px 0px 10px 0px;" width="640" /></p> 
<p>Errors can also happen when messages contain unexpected data. Kinesis Data Analytics for SQL Applications provides an in-application error stream. These errors can then be processed separately and restreamed or dropped. With PyFlink in Kinesis Data Analytics Streaming applications, you can write complex error-handling strategies and immediately recover and continue processing the data. When the JSON string is passed into the UDF, it may be malformed, incomplete, or empty. By catching the error in the UDF, Python will always return a value even if an error would have occurred.</p> 
<p>The following sample code shows another PyFlink snippet that performs a division calculation on two fields. If a division-by-zero error is encountered, it provides a default value so the stream can continue processing the message.</p> 
<pre><code class="lang-python">%flink.pyflink
@udf(input_types=[DataTypes.BIGINT()], result_type=DataTypes.BIGINT())
def DivideByZero(price):    
	try:        
		price / 0        
	except:        
		return -1
st_env.register_function("DivideByZero", DivideByZero)
</code></pre> 
<h2>Next steps</h2> 
<p>Building a pipeline as we’ve done in this post gives us the base for testing additional services in AWS. I encourage you to continue your streaming analytics learning before tearing down the streams you created. Consider the following:</p> 
<ul> 
 <li>Publish your Amazon Managed Service for Apache Flink Studio notebook as an <a href="https://docs.aws.amazon.com/kinesisanalytics/latest/java/how-notebook-durable.html" rel="noopener" target="_blank">application with durable state</a>.</li> 
 <li>Use your Kinesis Data Firehose delivery stream to write directly to OpenSearch Service.</li> 
 <li>Use <a href="https://docs.aws.amazon.com/opensearch-service/latest/developerguide/dashboards.html" rel="noopener" target="_blank">OpenSearch Dashboards</a> to visualize your streaming data.</li> 
 <li>Review the <a href="https://docs.aws.amazon.com/kinesisanalytics/latest/dev/examples-migrating-to-kda-studio.html" rel="noopener" target="_blank">Migrating to Amazon Managed Service for Apache Flink</a>: Examples docs for side-by-side translations of common SQL-based Kinesis Data Analytics application queries to Amazon Managed Service for Apache Flink Studio.</li> 
</ul> 
<h2>Clean up</h2> 
<p>To clean up the services created in this exercise, complete the following steps:</p> 
<ol> 
 <li>Navigate to the CloudFormation Console and delete the IoT Device Simulator stack.</li> 
 <li>On the AWS IoT Core console, choose Message Routing and Rules, and delete the rule <code>automotive_route_kinesis</code>.</li> 
 <li>Delete the Kinesis data stream <code>automotive-data</code> in the Kinesis Data Stream console.</li> 
 <li>Remove the IAM role <code>automotive-role</code> in the IAM Console.</li> 
 <li>In the AWS Glue console, delete the <code>automotive-notebook-glue</code> database.</li> 
 <li>Delete the Amazon Managed Service for Apache Flink Studio notebook <code>automotive-data-notebook</code>.</li> 
 <li>Delete the Firehose delivery stream <code>automotive-firehose</code>.</li> 
</ol> 
<h2>Conclusion</h2> 
<p>Thanks for following along with this tutorial on Amazon Managed Service for Apache Flink Studio. If you’re currently using a legacy Amazon Managed Service for Apache Flink Studio SQL application, I recommend you reach out to your AWS technical account manager or Solutions Architect and discuss migrating to Amazon Managed Service for Apache Flink Studio. You can continue your learning path in our <a href="https://docs.aws.amazon.com/streams/latest/dev/introduction.html" rel="noopener" target="_blank">Amazon Kinesis Data Streams Developer Guide</a>, and access our <a href="https://github.com/aws-samples/amazon-kinesis-data-analytics-examples" rel="noopener" target="_blank">code samples on GitHub</a>.</p> 
<hr /> 
<h3>About the Author</h3> 
<p><img alt="" class="size-full wp-image-38835 alignleft" height="100" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/11/29/Nic-sm-copy-1.png" width="100" /><strong>Nicholas Tunney</strong> is a Partner Solutions Architect for Worldwide Public Sector at AWS. He works with global SI partners to develop architectures on AWS for clients in the government, nonprofit healthcare, utility, and education sectors.</p>
