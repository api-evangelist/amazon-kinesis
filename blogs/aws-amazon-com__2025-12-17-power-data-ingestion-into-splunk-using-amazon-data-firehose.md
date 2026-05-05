---
title: "Power data ingestion into Splunk using Amazon Data Firehose"
url: "https://aws.amazon.com/blogs/big-data/power-data-ingestion-into-splunk-using-amazon-data-firehose/"
date: "Wed, 17 Dec 2025 18:52:29 +0000"
author: "Tarik Makota"
feed_url: "https://aws.amazon.com/blogs/big-data/category/analytics/amazon-kinesis/feed/"
---
<div class="content-freshness"> 
 <strong>Last updated:</strong> December 17, 2025
 <br /> 
 <em>Originally published: December 18, 2017</em> 
</div> 
<p><a href="https://aws.amazon.com/kinesis/data-firehose/" rel="noopener noreferrer" target="_blank">Amazon Data Firehose</a> supports Splunk Enterprise and Splunk Cloud as a delivery destination. This native integration between Splunk Enterprise, Splunk Cloud, and Amazon Data Firehose is designed to make AWS data ingestion setup seamless, while offering a secure and fault-tolerant delivery mechanism. We want to enable customers to monitor and analyze machine data from any source and use it to deliver operational intelligence and optimize IT, security, and business performance.</p> 
<p>With Amazon Data Firehose, customers can use a fully managed, reliable, and scalable data streaming solution to Splunk. In this post, we tell you a bit more about the Amazon Data Firehose and Splunk integration. We also show you how to ingest large amounts of data into Splunk using Amazon Data Firehose.</p> 
<h2>Push vs. Pull data ingestion</h2> 
<p>Presently, customers use a combination of two ingestion patterns, primarily based on data source and volume, in addition to existing company infrastructure and expertise:</p> 
<ol> 
 <li>Pull-based approach: Using dedicated pollers running the popular <a href="https://splunkbase.splunk.com/app/1876/" rel="noopener noreferrer" target="_blank">Splunk Add-on for AWS</a> to pull data from various AWS services such as <a href="https://aws.amazon.com/cloudwatch/" rel="noopener" target="_blank">Amazon CloudWatch</a> or <a href="https://aws.amazon.com/s3/" rel="noopener noreferrer" target="_blank">Amazon S3</a>.</li> 
 <li>Push-based approach: Streaming data directly from AWS to Splunk HTTP Event Collector (HEC) by using <a href="https://aws.amazon.com/kinesis/data-firehose/" rel="noopener" target="_blank">Amazon Data Firehose</a>. Examples of applicable data sources include CloudWatch Logs and <a href="https://aws.amazon.com/kinesis/data-streams/" rel="noopener" target="_blank">Amazon Kinesis Data Streams</a>.</li> 
</ol> 
<p>The pull-based approach offers data delivery guarantees such as retries and checkpointing out of the box. However, it requires more ops to manage and orchestrate the dedicated pollers, which are commonly running on <a href="https://aws.amazon.com/ec2/" rel="noopener noreferrer" target="_blank">Amazon EC2</a> instances. With this setup, you pay for the infrastructure even when it’s idle.</p> 
<p>On the other hand, the push-based approach offers a low-latency scalable data pipeline made up of serverless resources like Amazon Data Firehose sending directly to Splunk indexers (by using Splunk HEC). This approach translates into lower operational complexity and cost. However, if you need guaranteed data delivery then you have to design your solution to handle issues such as a Splunk connection failure or Lambda execution failure. To do so, you might use, for example, <a href="http://docs.aws.amazon.com/lambda/latest/dg/dlq.html" rel="noopener noreferrer" target="_blank">AWS Lambda Dead Letter Queues</a>.</p> 
<h2>How about getting the best of both worlds?</h2> 
<p>Let’s go over the new integration’s end-to-end solution and examine how Amazon Data Firehose and Splunk together expand the push-based approach into a native AWS solution for applicable data sources.</p> 
<p><img alt="" class="alignnone wp-image-82791 size-full" height="671" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/08/27/image-2.jpeg" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="641" /></p> 
<p>By using a managed service like Amazon Data Firehose for data ingestion into Splunk, we provide out-of-the-box reliability and scalability. One of the pain points of the old approach was the overhead of managing the data collection nodes (Splunk heavy forwarders). With the new Amazon Data Firehose to Splunk integration, there are no forwarders to manage or set up. Data producers (1) are configured through the AWS Management Console to drop data into Amazon Data Firehose.</p> 
<p>You can also create your own data producers. For example, you can drop data into a Firehose delivery stream by using <a href="http://docs.aws.amazon.com/firehose/latest/dev/writing-with-agents.html" rel="noopener noreferrer" target="_blank">Amazon Kinesis Agent</a>, or by using the Firehose API (<tt>PutRecord(), PutRecordBatch()</tt>), or by writing to a Kinesis Data Stream configured to be the data source of a Firehose delivery stream. For more details, refer to <a href="http://docs.aws.amazon.com/firehose/latest/dev/basic-write.html" rel="noopener noreferrer" target="_blank">Sending Data to an Amazon Data Firehose Delivery Stream</a><strong>.</strong></p> 
<p>You might need to transform the data before it goes into Splunk for analysis. For example, you might want to enrich it or filter or anonymize sensitive data. You can do so using AWS Lambda and enabling data transformation in Amazon Data Firehose. In this scenario, Amazon Data Firehose is used to decompress the Amazon CloudWatch logs by enabling the feature.</p> 
<p>Systems fail all the time. Let’s see how this integration handles outside failures to guarantee data durability. In cases when Amazon Data Firehose can’t deliver data to the Splunk Cluster, data is automatically backed up to an S3 bucket. You can configure this feature while creating the Firehose delivery stream (2). You can choose to back up all data or only the data that’s failed during delivery to Splunk.</p> 
<p>In addition to using S3 for data backup, this Firehose integration with Splunk supports Splunk Indexer Acknowledgments to guarantee event delivery. This feature is configured on Splunk’s HTTP Event Collector (HEC) (3). It ensures that HEC returns an acknowledgment to Amazon Data Firehose only after data has been indexed and is available in the Splunk cluster (4).</p> 
<p>Now let’s look at a hands-on exercise that shows how to forward VPC flow logs to Splunk.</p> 
<h2>How-to guide</h2> 
<p>To process VPC flow logs, we implement the following architecture.</p> 
<p><img alt="" class="alignnone size-full wp-image-82793" height="451" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/08/27/image-4.jpeg" width="1061" /></p> 
<p><a href="https://aws.amazon.com/vpc/" rel="noopener noreferrer" target="_blank">Amazon Virtual Private Cloud (Amazon VPC)</a> delivers flow log files into an Amazon CloudWatch Logs group. Using a CloudWatch Logs subscription filter, we set up real-time delivery of CloudWatch Logs to an Amazon Data Firehose stream.</p> 
<p>Data coming from CloudWatch Logs is compressed with gzip compression. To work with this compression, we will enable decompression for the Firehose stream. Firehose then delivers the raw logs to the Splunk Http Event Collector (HEC).</p> 
<p>If delivery to the Splunk HEC fails, Firehose deposits the logs into an Amazon S3 bucket. You can then ingest the events from S3 using an alternate mechanism such as a Lambda function.</p> 
<p>When data reaches Splunk (Enterprise or Cloud), Splunk parsing configurations (packaged in the Splunk Add-on for Amazon Data Firehose) extract and parse all fields. They make data ready for querying and visualization using Splunk Enterprise and Splunk Cloud.</p> 
<h2>Walkthrough</h2> 
<h3>Install the Splunk Add-on for Amazon Data Firehose</h3> 
<p>The Splunk Add-on for Amazon Data Firehose enables Splunk (be it Splunk Enterprise, Splunk App for AWS, or Splunk Enterprise Security) to use data ingested from Amazon Data Firehose. Install the Add-on on all the indexers with an HTTP Event Collector (HEC). The Add-on is available for download from <a href="https://splunkbase.splunk.com/app/3719/" rel="noopener noreferrer" target="_blank">Splunkbase</a>. For troubleshooting assistance, please refer to: <a href="https://docs.aws.amazon.com/firehose/latest/dev/data-not-delivered-to-splunk.html">AWS Data Firehose troubleshooting documentation</a> &amp;&nbsp;<a href="https://docs.splunk.com/Documentation/AddOns/released/Firehose/Troubleshoot">Splunk’s official troubleshooting guide</a></p> 
<h3>HTTP Event Collector (HEC)</h3> 
<p>Before you can use Amazon Data Firehose to deliver data to Splunk, set up the Splunk HEC to receive the data. From Splunk web, go to the <strong>Setting</strong> menu, choose <strong>Data Inputs</strong>, and choose <strong>HTTP Event Collector</strong>. Choose <strong>Global Settings</strong>, ensure <strong>All tokens</strong> is enabled, and then choose <strong>Save</strong>. Then choose <strong>New Token</strong> to create a new HEC endpoint and token. When you create a new token, make sure that <strong>Enable indexer acknowledgment</strong> is checked.</p> 
<p><img alt="" class="alignnone size-full wp-image-4076" height="309" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2017/12/15/splunk_kinesis3.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" /></p> 
<p>When prompted to select a source type, select <strong>aws:cloudwatchlogs:vpcflow</strong></p> 
<p><img alt="" class="alignnone size-full wp-image-4077" height="287" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2017/12/15/splunk_kinesis4.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" /></p> 
<h3>Create an S3 backsplash bucket</h3> 
<p>To provide for situations in which Amazon Data Firehose can’t deliver data to the Splunk Cluster, we use an S3 bucket to back up the data. You can configure this feature to back up all data or only the data that’s failed during delivery to Splunk.</p> 
<p><strong>Note:</strong> Bucket names are unique.</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">aws s3 create-bucket --bucket &lt;your-s3-bucket-name&gt; --create-bucket-configuration LocationConstraint=&lt;your-region&gt;</code></pre> 
</div> 
<h3>Create an Amazon Data Firehose delivery stream</h3> 
<p>On the AWS console, open the Amazon Data Firehose console, and choose <strong>Create Firehose Stream</strong>.</p> 
<p>Select DirectPUT as the source and Splunk as the destination.</p> 
<p><img alt="Create Firehose Stream" class="alignnone size-full wp-image-84723" height="539" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/10/16/Screenshot-2025-10-16-at-2.27.53 PM.png" width="1248" /></p> 
<p>If you are using Firehose to deliver CloudWatch Logs and want to deliver decompressed data to your Firehose stream destination, use Firehose Data Format Conversion (Parquet, ORC) or Dynamic partitioning. You must enable decompression for your Firehose stream, check out <a href="https://aws.amazon.com/blogs/big-data/deliver-decompressed-amazon-cloudwatch-logs-to-amazon-s3-and-splunk-using-amazon-data-firehose/" rel="noopener" target="_blank">Deliver decompressed Amazon CloudWatch Logs to Amazon S3 and Splunk using Amazon Data Firehose</a></p> 
<p><img alt="" class="alignnone size-full wp-image-82794" height="612" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/08/27/image-5.png" width="1431" /></p> 
<p>Enter your Splunk HTTP Event Collector (HEC) information in destination settings</p> 
<p><img alt="Firehose Destination setting" class="alignnone size-full wp-image-84731" height="874" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/10/16/Screenshot-2025-10-16-at-2.43.39 PM.png" width="1209" /></p> 
<p><strong>Note:</strong> Amazon Data Firehose requires the Splunk HTTP Event Collector (HEC) endpoint to be terminated with a valid CA-signed certificate matching the DNS hostname used to connect to your HEC endpoint. You receive delivery errors if you are using a self-signed certificate.</p> 
<p>In this example, we only back up logs that fail during delivery.</p> 
<p><img alt="Backsplash S3 settings" class="alignnone size-full wp-image-84728" height="575" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/10/16/Screenshot-2025-10-16-at-2.36.33 PM.png" width="1208" /></p> 
<p>To monitor your Firehose delivery stream, enable error logging. Doing this means that you can monitor record delivery errors. Create an IAM role for the Firehose stream by choosing Create new, or Choose existing IAM role.</p> 
<p><img alt="Advance settings for cloudwatch loggings" class="alignnone size-full wp-image-84730" height="704" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/10/16/Screenshot-2025-10-16-at-2.38.49 PM.png" width="1209" /></p> 
<p>You now get a chance to review and adjust the Firehose stream settings. When you are satisfied, choose <strong>Create Firehose Stream</strong>.</p> 
<h3>Create a VPC Flow Log</h3> 
<p>To send events from Amazon VPC, you need to set up a VPC flow log. If you already have a VPC flow log you want to use, you can skip to the “Publish CloudWatch to Amazon Data Firehose” section.</p> 
<p>On the AWS console, open the Amazon VPC service. Then choose <strong>VPC</strong>, and choose the VPC you want to send flow logs from. Choose <strong>Flow Logs</strong>, and then choose <strong>Create Flow Log</strong>. If you don’t have an IAM role that allows your VPC to publish logs to CloudWatch, choose <b>Create and use a new service role</b>.</p> 
<p><img alt="VPC Flow Logs Settings" class="alignnone size-full wp-image-84737" height="1014" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/10/16/Screenshot-2025-10-16-at-2.53.04 PM.png" width="1229" /></p> 
<p>Once active, your VPC flow log should look like the following.</p> 
<p><img alt="Flow logs" class="alignnone size-full wp-image-84738" height="168" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/10/16/Screenshot-2025-10-16-at-2.55.17 PM.png" width="1452" /></p> 
<h2>Publish CloudWatch to Amazon Data Firehose</h2> 
<p>When you generate traffic to or from your VPC, the log group is created in Amazon CloudWatch. We create an IAM role to allow Cloudwatch to publish logs to the Amazon Data Firehose Stream.</p> 
<p>To allow CloudWatch to publish to your Firehose stream, you need to give it permissions.</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">$ aws iam create-role --role-name CWLtoFirehoseRole --assume-role-policy-document file://TrustPolicyForCWLToFireHose.json</code></pre> 
</div> 
<p><strong><em><br /> </em></strong>Here is the content for <tt>TrustPolicyForCWLToFireHose.json</tt>.</p> 
<div class="hide-language"> 
 <pre><code class="lang-json">{
  "Statement": {
    "Effect": "Allow",
    "Principal": { "Service": "logs.us-east-1.amazonaws.com" },
    "Action": "sts:AssumeRole"
  }
}
</code></pre> 
</div> 
<p>Attach the policy to the newly created role.</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">$ aws iam put-role-policy 
    --role-name CWLtoFirehoseRole 
    --policy-name Permissions-Policy-For-CWL 
    --policy-document file://PermissionPolicyForCWLToFireHose.json</code></pre> 
</div> 
<p>Here is the content for <tt>PermissionPolicyForCWLToFireHose.json</tt>.</p> 
<div class="hide-language"> 
 <pre><code class="lang-json">{
    "Statement":[
      {
        "Effect":"Allow",
        "Action":["firehose:*"],
        "Resource":["arn:aws:firehose:us-east-1:YOUR-AWS-ACCT-NUM:deliverystream/FirehoseSplunkDeliveryStream"]
      },
      {
        "Effect":"Allow",
        "Action":["iam:PassRole"],
        "Resource":["arn:aws:iam::YOUR-AWS-ACCT-NUM:role/CWLtoFirehoseRole"]
      }
    ]
}</code></pre> 
</div> 
<p>The new log group has no subscription filter, so set up a subscription filter. Setting this up establishes a real-time data feed from the log group to your Firehose delivery stream. Select the VPC flow log and choose Actions. Then choose Subscription filters followed by Create Amazon Data Firehose subscription filter.</p> 
<p><img alt="Subscription Filter option" class="alignnone wp-image-84739 size-large" height="330" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/10/16/Screenshot-2025-10-16-at-2.59.44 PM-1024x330.png" width="1024" /></p> 
<p><img alt="Subscription filter details" class="alignnone wp-image-84741 size-full" height="997" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/10/16/Screenshot-2025-10-16-at-3.08.06 PM.png" width="743" /></p> 
<p>When you run the AWS CLI command preceding, you don’t get any acknowledgment. To validate that your CloudWatch Log Group is subscribed to your Firehose stream, check the CloudWatch console.</p> 
<p>As soon as the subscription filter is created, the real-time log data from the log group goes into your Firehose delivery stream. Your stream then delivers it to your Splunk Enterprise or Splunk Cloud environment for querying and visualization. The screenshot following is from Splunk Enterprise.</p> 
<p><img alt="" class="alignnone size-full wp-image-4094" height="284" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2017/12/15/splunk_kinesis21.png" style="margin: 20px 0px 20px 0px; border: 1px solid #cccccc;" width="800" /></p> 
<p>In addition, you can monitor and view metrics associated with your delivery stream using the AWS console.</p> 
<p><img alt="" class="alignnone size-full wp-image-84743" height="484" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/10/16/Screenshot-2025-10-16-at-3.13.42 PM.png" width="1476" /></p> 
<h2>Conclusion</h2> 
<p>Although our walkthrough uses VPC Flow Logs, the pattern can be used in many other scenarios. These include ingesting data from AWS IoT, other CloudWatch logs and events, Kinesis Streams or other data sources using the Kinesis Agent or Kinesis Producer Library. You may use a Lambda blueprint or disable record transformation entirely depending on your use case. For an additional use case using Amazon Data Firehose, check out <a href="https://www.youtube.com/watch?v=EkY0P0UkIIg" rel="noopener noreferrer" target="_blank">This is My Architecture Video</a>, which discusses how to securely centralize cross-account data analytics using Kinesis and Splunk.</p> 
<p>If you found this post useful, be sure to check out <a href="https://aws.amazon.com/blogs/big-data/integrating-splunk-with-amazon-kinesis-streams/" rel="noopener noreferrer" target="_blank">Integrating Splunk with Amazon Kinesis Streams</a>.</p> 
<hr /> 
<h3>About the Authors</h3> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Tarik Makota" class="aligncenter size-full wp-image-29797" height="160" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2017/12/18/Tarik_1.jpg" width="120" />
  </div> 
  <h3 class="lb-h4">Tarik Makota</h3> 
  <p>Tarik is a solutions architect with the Amazon Web Services Partner Network. He provides technical guidance, design advice and thought leadership to AWS’ most strategic software partners. His career includes work in an extremely broad software development and architecture roles across ERP, financial printing, benefit delivery and administration and financial services. He holds an M.S. in Software Development and Management from Rochester Institute of Technology.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Roy Arsan" class="aligncenter size-full wp-image-29797" height="160" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2017/12/18/RoyArsan_1.jpg" width="120" />
  </div> 
  <h3 class="lb-h4">Roy Arsan</h3> 
  <p>Roy is a solutions architect in the Splunk Partner Integrations team. He has a background in product development, cloud architecture, and building consumer and enterprise cloud applications. More recently, he has architected Splunk solutions on major cloud providers, including an AWS Quick Start for Splunk that enables AWS users to easily deploy distributed Splunk Enterprise straight from their AWS console. He’s also the co-author of the AWS Lambda blueprints for Splunk. He holds an M.S. in Computer Science Engineering from the University of Michigan.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Yashika Jain" class="aligncenter size-full wp-image-29797" height="160" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/08/06/badgephotos.corp_.amazon.jpg" width="120" />
  </div> 
  <h3 class="lb-h4">Yashika Jain</h3> 
  <p>Yashika is a Senior Cloud Analytics Engineer at AWS, specializing&nbsp;in real-time analytics and event-driven architectures. She is committed to helping customers by providing deep technical guidance, driving best practices across real-time data platforms and solving complex issues related to their streaming data architectures.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Mitali Sheth" class="aligncenter size-full wp-image-29797" height="160" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/10/16/190012_Original-1-e1760653704300-100x118.jpg" width="120" />
  </div> 
  <h3 class="lb-h4">Mitali Sheth</h3> 
  <p>Mitali is a Streaming Data Engineer in the AWS Professional Services team, specializing in real-time analytics and event-driven architectures for AWS’ most strategic software customers. More recently, she has focused on data governance with AWS Lake Formation, building reliable data pipelines with AWS Glue, and modernizing streaming infrastructure with Amazon MSK and Amazon Managed Flink for large-scale enterprise deployments. She holds an M.S. in Computer Science from the University of Florida.</p> 
 </div> 
 <p><img alt="" class="wp-image-84750 size-thumbnail aligncenter" height="118" width="100" /></p> 
</footer>
