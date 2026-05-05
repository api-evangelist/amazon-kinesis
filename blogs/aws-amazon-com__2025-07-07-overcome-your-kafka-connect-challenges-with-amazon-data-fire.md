---
title: "Overcome your Kafka Connect challenges with Amazon Data Firehose"
url: "https://aws.amazon.com/blogs/big-data/overcome-your-kafka-connect-challenges-with-amazon-data-firehose/"
date: "Mon, 07 Jul 2025 14:26:11 +0000"
author: "Swapna Bandla"
feed_url: "https://aws.amazon.com/blogs/big-data/category/analytics/amazon-kinesis/feed/"
---
<p>Apache Kafka is a popular open source distributed streaming platform that is widely used in the AWS ecosystem. It’s designed to handle real-time, high-throughput data streams, making it well-suited for building real-time data pipelines to meet the streaming needs of modern cloud-based applications.</p> 
<p>For AWS customers looking to run Apache Kafka, but don’t want to worry about the undifferentiated heavy lifting involved with self-managing their Kafka clusters, <a href="https://aws.amazon.com/msk/" rel="noopener" target="_blank">Amazon Managed Streaming for Apache Kafka</a> (Amazon MSK) offers fully managed Apache Kafka. This means Amazon MSK provisions your servers, configures your Kafka clusters, replaces servers when they fail, orchestrates server patches and upgrades, makes sure clusters are architected for high availability, makes sure data is durably stored and secured, sets up monitoring and alarms, and runs scaling to support load changes. With a managed service, you can spend your time developing and running streaming event applications.</p> 
<p>For applications to use data sent to Kafka, you need to write, deploy, and manage application code that consumes data from Kafka.</p> 
<p>Kafka Connect is an open-source component of the Kafka project that provides a framework for connecting with external systems such as databases, key-value stores, search indexes, and file systems from your Kafka clusters. On AWS, our customers commonly write and manage connectors using the Kafka Connect framework to move data out of their Kafka clusters into persistent storage, like <a href="http://aws.amazon.com/s3" rel="noopener" target="_blank">Amazon Simple Storage Service</a> (Amazon S3), for long-term storage and historical analysis.</p> 
<p>At scale, customers need to programmatically manage their Kafka Connect infrastructure for consistent deployments when updates are required, as well as the code for error handling, retries, compression, or data transformation as it is delivered from your Kafka cluster. However, this introduces a need for investment into the <a href="https://aws.amazon.com/what-is/sdlc/" rel="noopener" target="_blank">software development lifecycle</a> (SDLC) of this management software. Although the SDLC is a cost-effective and time-efficient process to help development teams build high-quality software, for many customers, this process is not desirable for their data delivery use case, particularly when they could dedicate more resources towards innovating for other key business differentiators. Beyond SDLC challenges, many customers face fluctuating data streaming throughput. For instance:</p> 
<ul> 
 <li>Online gaming businesses experience throughput variations based on game usage</li> 
 <li>Video streaming applications see changes in throughput depending on viewership</li> 
 <li>Traditional businesses have throughput fluctuations tied to consumer activity</li> 
</ul> 
<p>Striking the right balance between resources and workload can be challenging. Under-provisioning can lead to consumer lag, processing delays, and potential data loss during peak loads, hampering real-time data flows and business operations. On the other hand, over-provisioning results in underutilized resources and unnecessary high costs, making the setup economically inefficient for customers. Even the action of scaling up your infrastructure introduces additional delays because resources need to be provisioned and acquired for your Kafka Connect cluster.</p> 
<p>Even when you can estimate aggregated throughput, predicting throughput per individual stream remains difficult. As a result, to achieve smooth operations, you might resort to over-provisioning your Kafka Connect resources (CPU) for your streams. This approach, though functional, might not be the most efficient or cost-effective solution.</p> 
<p>Customers have been asking for a fully serverless solution that will not only handle managing resource allocation, but transition the cost model to only pay for the data they are delivering from the Kafka topic, instead of underlying resources that require constant monitoring and management.</p> 
<p>In September 2023, we announced a new integration between Amazon MSK and <a href="https://aws.amazon.com/firehose/" rel="noopener" target="_blank">Amazon Data Firehose</a>, allowing builders to deliver data from their MSK topics to their destination sinks with a fully managed, serverless solution. With this new integration, you no longer needed to develop and manage your own code to read, transform, and write your data to your sink using Kafka Connect. Firehose abstracts away the retry logic required when reading data from your MSK cluster and delivering it to the desired sink, as well as infrastructure provisioning, because it can scale out and scale in automatically to adjust to the volume of data to transfer. There are no provisioning or maintenance operations required on your side.</p> 
<p>At release, the checkpoint time to start consuming data from the MSK topic was the creation time of the Firehose stream. Firehose couldn’t start reading from other points on the data stream. This caused challenges for several different use cases.</p> 
<p>For customers that are setting up a mechanism to sink data from their cluster for the first time, all data in the topic older than the timestamp of Firehose stream creation would need another way to be persisted. For example, customers using Kafka Connect connectors, like These users were limited in using Firehose because they wanted to sink all the data from the topic to their sink, but Firehose couldn’t read data from earlier than the timestamp of Firehose stream creation.</p> 
<p>For other customers that were running Kafka Connect and needed to migrate from their Kafka Connect infrastructure to Firehose, this required some extra coordination. The release functionality of Firehose means you can’t point your Firehose stream to a specific point on the source topic, so a migration requires stopping data ingest to the source MSK topic and waiting for Kafka Connect to sink all the data to the destination. Then you can create the Firehose stream and restart the producers such that the Firehose stream can then consume new messages from the topic. This adds additional, and non-trivial, overhead to the migration effort when attempting to cut over from an existing Kafka Connect infrastructure to a new Firehose stream.</p> 
<p>To address these challenges, we’re happy to announce a new feature in the Firehose integration with Amazon MSK. You can now specify the Firehose stream to either read from the earliest position on the Kafka topic or from a custom timestamp to begin reading from your MSK topic.</p> 
<p>In the <a href="https://aws.amazon.com/blogs/aws/amazon-msk-introduces-managed-data-delivery-from-apache-kafka-to-your-data-lake/" rel="noopener" target="_blank">first post of this series</a>, we focused on managed data delivery from Kafka to your data lake. In this post, we extend the solution to choose a custom timestamp for your MSK topic to be synced to Amazon S3.</p> 
<h2>Overview of Firehose integration with Amazon MSK</h2> 
<p>Firehose integrates with Amazon MSK to offer a fully managed solution that simplifies the processing and delivery of streaming data from Kafka clusters into data lakes stored on Amazon S3. With just a few clicks, you can continuously load data from your desired Kafka clusters to an S3 bucket in the same account, eliminating the need to develop or run your own connector applications. The following are some of the key benefits to this approach:</p> 
<ul> 
 <li><strong>Fully managed service</strong> – Firehose is a fully managed service that handles the provisioning, scaling, and operational tasks, allowing you to focus on configuring the data delivery pipeline.</li> 
 <li><strong>Simplified configuration</strong> – With Firehose, you can set up the data delivery pipeline from Amazon MSK to your sink with just a few clicks on the <a href="http://aws.amazon.com/console" rel="noopener" target="_blank">AWS Management Console</a>.</li> 
 <li><strong>Automatic scaling</strong> – Firehose automatically scales to match the throughput of your Amazon MSK data, without the need for ongoing administration.</li> 
 <li><strong>Data transformation and optimization</strong> – Firehose offers features like JSON to Parquet/ORC conversion and batch aggregation to optimize the delivered file size, simplifying data analytical processing workflows.</li> 
 <li><strong>Error handling and retries</strong> – Firehose automatically retries data delivery in case of failures, with configurable retry durations and backup options.</li> 
 <li><strong>Offset select option</strong> – With Firehose, you can select the starting position for the MSK delivery stream to be delivered within a topic from three options: 
  <ul> 
   <li><strong>Firehose stream creation time</strong> – This allows you to deliver data starting from Firehose stream creation time. When migrating from to Firehose, if you have an option to pause the producer, you can consider this option.</li> 
   <li><strong>Earliest</strong> – This allows you to deliver data starting from MSK topic creation time. You can choose this option if you’re setting a new delivery pipeline with Firehose from Amazon MSK to Amazon S3.</li> 
   <li><strong>At timestamp</strong> – This option allows you to provide a specific start date and time in the topic from where you want the Firehose stream to read data. The time is in your local time zone. You can choose this option if you prefer not to stop your producer applications while migrating from Kafka Connect to Firehose. You can refer to the Python script and steps provided later in this post to derive the timestamp for the latest events in your topic that were consumed by Kafka Connect.</li> 
  </ul> </li> 
</ul> 
<p><img alt="" class="alignnone size-full wp-image-69005" height="719" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/09/13/BDB-4358-image001.png" style="margin: 10px 0px 10px 0px;" width="1168" /></p> 
<p>The following are benefits of the new timestamp selection feature with Firehose:</p> 
<ul> 
 <li>You can select the starting position of the MSK topic, not just from the point that the Firehose stream is created, but from any point from the earliest timestamp of the topic.</li> 
 <li>You can replay the MSK stream delivery if required, for example in the case of testing scenarios to select from different timestamps with the option to select from a specific timestamp.</li> 
 <li>When migrating from Kafka Connect to Firehose, gaps or duplicates can be managed by selecting the starting timestamp for Firehose delivery from the point where Kafka Connect delivery ended. Because the new custom timestamp feature isn’t monitoring Kafka consumer offsets per partition, the timestamp you select for your Kafka topic should be a few minutes before the timestamp at which you stopped Kafka Connect. The earlier the timestamp you select, the more duplicate records you will have downstream. The closer the timestamp to the time of Kafka Connect stopping, the higher the likelihood of data loss if certain partitions have fallen behind. Be sure to select a timestamp appropriate to your requirements.</li> 
</ul> 
<h2>Overview of solution</h2> 
<p>We discuss two scenarios to stream data.</p> 
<p>In Scenario 1, we migrate to Firehose from Kafka Connect with the following steps:</p> 
<ol> 
 <li>Derive the latest timestamp from MSK events that Kafka Connect delivered to Amazon S3.</li> 
 <li>Create a Firehose delivery stream with Amazon MSK as the source and Amazon S3 as the destination with the topic starting position as <strong>Earliest</strong>.</li> 
 <li>Query Amazon S3 to validate the data loaded.</li> 
</ol> 
<p>In Scenario 2, we create a new data pipeline from Amazon MSK to Amazon S3 with Firehose:</p> 
<ol> 
 <li>Create a Firehose delivery stream with Amazon MSK as the source and Amazon S3 as the destination with the topic starting position as <strong>At timestamp</strong>.</li> 
 <li>Query Amazon S3 to validate the data loaded.</li> 
</ol> 
<p>The solution architecture is depicted in the following diagram.</p> 
<p><img alt="" class="alignnone size-full wp-image-69006" height="372" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/09/13/BDB-4358-image003.png" style="margin: 10px 0px 10px 0px;" width="1019" /></p> 
<h2>Prerequisites</h2> 
<p>You should have the following prerequisites:</p> 
<ul> 
 <li>An <a href="https://portal.aws.amazon.com/billing/signup/iam?nc2=h_ct&amp;redirect_url=https%3A%2F%2Faws.amazon.com%2Fregistration-confirmation&amp;src=header_signup#/support" rel="noopener" target="_blank">AWS account</a> and access to the following AWS services: 
  <ul> 
   <li><a href="https://aws.amazon.com/ec2/" rel="noopener" target="_blank">Amazon Elastic Compute Cloud</a> (Amazon EC2)</li> 
   <li>Amazon Data Firehose</li> 
   <li><a href="https://aws.amazon.com/iam/" rel="noopener" target="_blank">AWS Identity and Access Management</a> (IAM)</li> 
   <li>Amazon MSK</li> 
   <li>Amazon S3</li> 
  </ul> </li> 
 <li>An MSK provisioned or MSK serverless cluster with topics created and data streaming to it. The sample topic used in this is <code>order</code>.</li> 
 <li>An EC2 instance configured to use as a Kafka admin client. Refer to <a href="https://docs.aws.amazon.com/msk/latest/developerguide/create-iam-role.html" rel="noopener" target="_blank">Create an IAM role</a> for instructions to create the client machine and IAM role that you will need to run commands against your MSK cluster.</li> 
 <li>An S3 bucket for delivering data from Amazon MSK using Firehose.</li> 
 <li>Kafka Connect to deliver data from Amazon MSK to Amazon S3 if you want to migrate from Kafka Connect (Scenario 1).</li> 
</ul> 
<h2>Migrate to Firehose from Kafka Connect</h2> 
<p>To reduce duplicates and minimize data loss, you need to configure your custom timestamp for Firehose to read events as close to the timestamp of the oldest committed offset that Kafka Connect reported. You can follow the steps in this section to visualize how the timestamps of each committed offset will vary by partition across the topic you want to read from. This is for demonstration purposes and doesn’t scale as a solution for workloads with a large number of partitions.</p> 
<p>Sample data was generated for demonstration purposes by following the instructions referenced in the following <a href="https://github.com/aws-samples/clickstream-producer-for-apache-kafka" rel="noopener" target="_blank">GitHub repo</a>. We set up a sample producer application that generates clickstream events to simulate users browsing and performing actions on an imaginary ecommerce website.</p> 
<p>To derive the latest timestamp from MSK events that Kafka Connect delivered to Amazon S3, complete the following steps:</p> 
<ol> 
 <li>From your Kafka client, query Amazon MSK to retrieve the Kafka Connect consumer group ID: 
  <div class="hide-language"> 
   <pre><code class="lang-bash">./kafka-consumer-groups.sh --bootstrap-server $bs --list --command-config client.properties</code></pre> 
  </div> <p><img alt="" class="alignnone size-full wp-image-69008" height="103" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/09/13/BDB-4358-image007.png" style="margin: 10px 0px 10px 0px;" width="1347" /></p></li> 
 <li>Stop Kafka Connect.</li> 
 <li>Query Amazon MSK for the latest offset and associated timestamp for the consumer group belonging to Kafka Connect.</li> 
</ol> 
<p>You can use the <code>get_latest_offsets.py</code> Python script from the following <a href="https://github.com/aws-samples/How-to-Overcome-your-Kafka-Connect-Challenges-with-Amazon-Data-Firehose" rel="noopener" target="_blank">GitHub repo</a> as a reference to get the timestamp associated with the latest offsets for your Kafka Connect consumer group. To enable authentication and authorization for a non-Java client with an IAM authenticated MSK cluster, refer to the following <a href="https://github.com/aws/aws-msk-iam-sasl-signer-python/blob/main/README.rst" rel="noopener" target="_blank">GitHub repo</a> for instructions on installing the <code>aws-msk-iam-sasl-signer-python</code> package for your client.</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">python3 get_latest_offsets.py --broker-list $bs --topic-name “order” --consumer-group-id “connect-msk-serverless-connector-090224” --aws-region “eu-west-1”</code></pre> 
</div> 
<p><img alt="" class="alignnone size-full wp-image-69009" height="299" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/09/13/BDB-4358-image009.png" style="margin: 10px 0px 10px 0px;" width="1347" /></p> 
<p>Note the earliest timestamp across all the partitions.</p> 
<h2>Create a data pipeline from Amazon MSK to Amazon S3 with Firehose</h2> 
<p>The steps in this section are applicable to both scenarios. Complete the following steps to create your data pipeline:</p> 
<ol> 
 <li>On the Firehose console, choose <strong>Firehose streams </strong>in the navigation pane.</li> 
 <li>Choose <strong>Create Firehose stream</strong>.<br /> <img alt="" class="alignnone size-full wp-image-69010" height="176" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/09/13/BDB-4358-image011.jpg" style="margin: 10px 0px 10px 0px;" width="1435" /></li> 
 <li>For <strong>Source</strong>, choose <strong>Amazon MSK</strong>.</li> 
 <li>For <strong>Destination</strong>, choose <strong>Amazon S3</strong>.<br /> <img alt="" class="alignnone size-full wp-image-69011" height="462" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/09/13/BDB-4358-image013.jpg" style="margin: 10px 0px 10px 0px;" width="813" /></li> 
 <li>For <strong>Source settings</strong>, browse to the MSK cluster and enter the topic name you created as part of the prerequisites.</li> 
 <li>Configure the Firehose stream starting position based on your scenario: 
  <ol type="a"> 
   <li>For Scenario 1, set <strong>Topic starting position</strong> as <strong>At Timestamp</strong> and enter the timestamp you noted in the previous section.<br /> <img alt="" class="alignnone size-full wp-image-69012" height="1004" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/09/13/BDB-4358-image015.png" style="margin: 10px 0px 10px 0px;" width="1167" /></li> 
   <li>For Scenario 2, set <strong>Topic starting position</strong> as <strong>Earliest</strong>.<br /> <img alt="" class="alignnone size-full wp-image-69013" height="876" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/09/13/BDB-4358-image017.png" style="margin: 10px 0px 10px 0px;" width="1166" /></li> 
  </ol> </li> 
 <li>For <strong>Firehose stream name</strong>, leave the default generated name or enter a name of your preference.</li> 
 <li>For <strong>Destination settings</strong>, browse to the S3 bucket created as part of the prerequisites to stream data.</li> 
</ol> 
<p>Within this S3 bucket, by default, a folder structure with <code>YYYY/MM/dd/HH</code> will be automatically created. Data will be delivered to subfolders pertaining to the HH subfolder according to the Firehose to Amazon S3 ingestion timestamp.</p> 
<p><img alt="" class="alignnone size-full wp-image-69014" height="191" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/09/13/BDB-4358-image019.jpg" style="margin: 10px 0px 10px 0px;" width="804" /></p> 
<ol start="9"> 
 <li>Under <strong>Advanced settings</strong>, you can choose to create the default IAM role for all the permissions that Firehose needs or choose existing an IAM role that has the policies that Firehose needs.<br /> <img alt="" class="alignnone size-full wp-image-69015" height="647" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/09/13/BDB-4358-image021.png" style="margin: 10px 0px 10px 0px;" width="768" /></li> 
 <li>Choose <strong>Create Firehose stream</strong>.<br /> <img alt="" class="alignnone size-full wp-image-69016" height="161" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/09/13/BDB-4358-image023.jpg" style="margin: 10px 0px 10px 0px;" width="808" /></li> 
</ol> 
<p>On the Amazon S3 console, you can verify the data streamed to the S3 folder according to your chosen offset settings.</p> 
<p><img alt="" class="alignnone size-full wp-image-69017" height="840" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/09/13/BDB-4358-image025.png" style="margin: 10px 0px 10px 0px;" width="1281" /></p> 
<h2>Clean up</h2> 
<p>To avoid incurring future charges, delete the resources you created as part of this exercise if you’re not planning to use them further.</p> 
<h2>Conclusion</h2> 
<p>Firehose provides a straightforward way to deliver data from Amazon MSK to Amazon S3, enabling you to save costs and reduce latency to seconds. To try Firehose with Amazon S3, refer to the <a href="https://catalog.workshops.aws/msk-labs/en-US/amazon-data-firehose-integration" rel="noopener" target="_blank">Delivery to Amazon S3 using Amazon Data Firehose</a> lab.</p> 
<hr /> 
<h3>About the Authors</h3> 
<p style="clear: both;"><img alt="" class="alignleft wp-image-69060 size-full" height="136" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/09/17/swapna.png" width="100" /><strong>Swapna Bandla</strong>&nbsp;is a Senior Solutions Architect in the AWS Analytics Specialist SA Team. Swapna has a passion towards understanding customers data and analytics needs and empowering them to develop cloud-based well-architected solutions. Outside of work, she enjoys spending time with her family.</p> 
<p style="clear: both;"><img alt="" class="alignleft size-full wp-image-69058" height="133" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2024/09/17/Austin.png" width="100" /><strong>Austin Groeneveld</strong> is a Streaming Specialist Solutions Architect at Amazon Web Services (AWS), based in the San Francisco Bay Area. In this role, Austin is passionate about helping customers accelerate insights from their data using the AWS platform. He is particularly fascinated by the growing role that data streaming plays in driving innovation in the data analytics space. Outside of his work at AWS, Austin enjoys watching and playing soccer, traveling, and spending quality time with his family.</p>
