---
title: "Accelerate data insights with Elastic and Amazon Kinesis Data Firehose"
url: "https://aws.amazon.com/blogs/big-data/accelerate-data-insights-with-elastic-and-amazon-kinesis-data-firehose/"
date: "Wed, 15 Mar 2023 18:16:05 +0000"
author: "Udayasimha Theepireddy"
feed_url: "https://aws.amazon.com/blogs/big-data/category/analytics/amazon-kinesis/kinesis-data-firehose/feed/"
---
<p><em><strong>February 9, 2024:</strong> Amazon Kinesis Data Firehose has been renamed to Amazon Data Firehose. Read the <a href="https://aws.amazon.com/about-aws/whats-new/2024/02/amazon-data-firehose-formerly-kinesis-data-firehose/" rel="noopener" target="_blank">AWS What’s New post</a> to learn more.</em></p> 
<p><em>This is a guest post co-written with Udayasimha Theepireddy from Elastic.</em></p> 
<p>Processing and analyzing log and Internet of Things (IoT) data can be challenging, especially when dealing with large volumes of real-time data. Elastic and <a href="https://aws.amazon.com/kinesis/data-firehose/" rel="noopener" target="_blank">Amazon Kinesis Data Firehose</a> are two powerful tools that can help make this process easier. For example, by using Kinesis Data Firehose to ingest data from IoT devices, you can stream data directly into Elastic for real-time analysis. This can help you identify patterns and anomalies in the data as they happen, allowing you to take action in real time. Additionally, by using Elastic to store and analyze log data, you can quickly search and filter through large volumes of log data to identify issues and troubleshoot problems.</p> 
<p>In this post, we explore how to integrate Elastic and Kinesis Data Firehose to streamline log and IoT data processing and analysis. We walk you through a step-by-step example of how to send <a href="https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html" rel="noopener" target="_blank">VPC flow logs</a> to Elastic through Kinesis Data Firehose.</p> 
<h2>Solution overview</h2> 
<p>Elastic is an <a href="https://partners.amazonaws.com/partners/001E000001A1vNOIAZ/Elastic" rel="noopener" target="_blank">AWS ISV Partner</a> that helps you find information, gain insights, and protect your data when you run on AWS. Elastic offers enterprise search, observability, and security features that are built on a single, flexible technology stack that can be deployed anywhere.</p> 
<p>Kinesis Data Firehose is a popular service that delivers streaming data from over 20 AWS services such as <a href="https://aws.amazon.com/iot-core/" rel="noopener" target="_blank">AWS IoT Core</a> and <a href="http://aws.amazon.com/cloudwatch" rel="noopener" target="_blank">Amazon CloudWatch</a> logs to over 15 analytical and observability tools such as Elastic. Kinesis Data Firehose provides a fast and easy way to send your VPC flow logs data to Elastic in minutes without a single line of code and without building or managing your own data ingestion and delivery infrastructure.</p> 
<p>VPC flow logs capture the traffic information going to and from your network interfaces in your VPC. With the launch of Kinesis Data Firehose support to Elastic, you can analyze your VPC flow logs with just a few clicks. Kinesis Data Firehose provides a true end-to-end serverless mechanism to deliver your flow logs to Elastic, where you can use Elastic Dashboards to search through those logs, create dashboards, detect anomalies, and send alerts. VPC flow logs help you to answer questions like what percentage of your traffic is getting dropped, and how much traffic is getting generated for specific sources and destinations.</p> 
<p>Integrating Elastic and Kinesis Data Firehose is a straightforward process. There are no agents and beats. Simply configure your Firehose delivery stream to send its data to Elastic’s endpoint.</p> 
<p>The following diagram depicts this specific configuration of how to ingest VPC flow logs via Kinesis Data Firehose into Elastic.</p> 
<p><img alt="" class="alignnone size-full wp-image-44521" height="356" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/03/13/bdb3086-elastic-kinesis-firehose-1.jpg" style="border: 1px solid #cccccc; margin: 10px 0px 10px 0px;" width="852" /></p> 
<p>In the past, users would have to use an <a href="Prerequisites" rel="noopener" target="_blank">AWS Lambda</a> function to transform the incoming data from VPC flow logs into an <a href="http://aws.amazon.com/s3" rel="noopener" target="_blank">Amazon Simple Storage Service</a> (Amazon S3) bucket before loading it into Kinesis Data Firehose or create a CloudWatch Logs subscription that sends any incoming log events that match defined filters to the Firehose delivery stream.</p> 
<p>With this new integration, you can set up this configuration directly from your VPC flow logs to Kinesis Data Firehose and into Elastic Cloud. (Note that Elastic Cloud must be deployed on AWS.)</p> 
<p>Let’s walk through the details of configuring Kinesis Data Firehose and Elastic, and demonstrate ingesting data.</p> 
<h2>Prerequisites</h2> 
<p>To set up this demonstration, make sure you have the following prerequisites:</p> 
<ul> 
 <li>An account on <a href="http://cloud.elastic.co" rel="noopener" target="_blank">Elastic Cloud</a> and a deployed stack on AWS. Deploying this on AWS is required for Kinesis Data Firehose log ingestion. For instructions, refer to <a href="https://www.elastic.co/guide/en/elastic-stack/current/installing-elastic-stack.html" rel="noopener" target="_blank">Installing the Elastic Stack</a>.</li> 
 <li>An AWS account with <a href="https://docs.elastic.co/en/integrations/aws#aws-permissions" rel="noopener" target="_blank">permissions</a> to pull the necessary data from AWS.</li> 
 <li>VPC flow logs enabled for the VPC where the application is deployed and configured to send data to Kinesis Data Firehose.</li> 
 <li>A <a href="https://github.com/aws-samples/aws-three-tier-web-architecture-workshop" rel="noopener" target="_blank">three-tier web architecture in AWS</a>, which can <a href="https://www.elastic.co/blog/aws-service-metrics-monitor-observability-easy" rel="noopener" target="_blank">ingest metrics from several AWS services</a>.</li> 
</ul> 
<p>We walk through installing general AWS integration components into the Elastic Cloud deployment to ensure Kinesis Data Firehose connectivity. Refer to the <a href="https://docs.elastic.co/en/integrations/aws#reference" rel="noopener" target="_blank">full list of services supported by the Elastic/AWS integration</a> for more information.</p> 
<h2>Deploy Elastic on AWS</h2> 
<p>Follow the instructions on the Elastic registration page to <a href="https://cloud.elastic.co/registration?fromURI=%2Fhome" rel="noopener" target="_blank">get started on Elastic Cloud</a>.</p> 
<p><img alt="" class="alignnone size-full wp-image-44522" height="444" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/03/13/bdb3086-elastic-kinesis-firehose-2.jpg" style="border: 1px solid #cccccc; margin: 10px 0px 10px 0px;" width="600" /></p> 
<p>Once logged in to Elastic Cloud, create a deployment on AWS. It’s important to ensure that the deployment is on AWS. The Firehose delivery stream connects specifically to an endpoint that needs to be on AWS.</p> 
<p><img alt="" class="alignnone size-full wp-image-44523" height="949" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/03/13/bdb3086-elastic-kinesis-firehose-3.jpg" style="border: 1px solid #cccccc; margin: 10px 0px 10px 0px;" width="1200" /></p> 
<p>After you create your deployment, copy the Elasticsearch endpoint to use in a later step.</p> 
<p><img alt="" class="alignnone size-full wp-image-44524" height="926" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/03/13/bdb3086-elastic-kinesis-firehose-4.jpg" style="border: 1px solid #cccccc; margin: 10px 0px 10px 0px;" width="1600" /></p> 
<p>The endpoint should be an AWS endpoint, such as <code>https://thevaa-cluster-01.es.us-east-1.aws.found.io</code>.</p> 
<h2>Enable Elastic’s AWS integration</h2> 
<p>In your deployment’s <strong>Elastic Integration</strong> section, navigate to the AWS integration and choose <strong>Install AWS assets</strong>.</p> 
<p><img alt="" class="alignnone size-full wp-image-44525" height="273" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/03/13/bdb3086-elastic-kinesis-firehose-5.jpg" style="border: 1px solid #cccccc; margin: 10px 0px 10px 0px;" width="800" /></p> 
<h2>Configure a Firehose delivery stream</h2> 
<p>Create a new delivery stream on the Kinesis Data Firehose console. This is where you provide the endpoint you saved earlier. Refer to the following screenshot for the destination settings, and for more details, refer to <a href="https://docs.aws.amazon.com/firehose/latest/dev/create-destination.html#create-destination-elastic" rel="noopener" target="_blank">Choose Elastic for Your Destination</a>.</p> 
<p><img alt="" class="alignnone size-full wp-image-44526" height="680" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/03/13/bdb3086-elastic-kinesis-firehose-6.jpg" style="border: 1px solid #cccccc; margin: 10px 0px 10px 0px;" width="600" /></p> 
<p>In this example, we are pulling in VPC flow logs via the data stream parameter we added (<code>logs-aws.vpcflow-default</code>). The parameter <code>es_datastream_name</code> can be configured with one of the following types of logs:</p> 
<ul> 
 <li><strong>logs-aws.cloudfront_logs-default</strong> – <a href="https://aws.amazon.com/cloudfront/" rel="noopener" target="_blank">AWS CloudFront</a> logs</li> 
 <li><strong>logs-aws.ec2_logs-default</strong> – <a href="http://aws.amazon.com/ec2" rel="noopener" target="_blank">Amazon Elastic Compute Cloud</a> (Amazon EC2) logs in CloudWatch</li> 
 <li><strong>logs-aws.elb_logs-default</strong> – <a href="https://aws.amazon.com/elasticloadbalancing/" rel="noopener" target="_blank">Elastic Load Balancing</a> logs</li> 
 <li><strong>logs-aws.firewall_logs-default</strong> – <a href="https://aws.amazon.com/network-firewall/" rel="noopener" target="_blank">AWS Network Firewall</a> logs</li> 
 <li><strong>logs-aws.route53_public_logs-default</strong> – <a href="https://aws.amazon.com/route53/" rel="noopener" target="_blank">Amazon Route 53</a> public DNS queries logs</li> 
 <li><strong>logs-aws.route53_resolver_logs-default</strong> – Route 53 DNS queries and responses logs</li> 
 <li><strong>logs-aws.s3access-default</strong> – Amazon S3 server access log</li> 
 <li><strong>logs-aws.vpcflow-default</strong> – VPC flow logs</li> 
 <li><strong>logs-aws.waf-default</strong> – <a href="https://aws.amazon.com/waf/" rel="noopener" target="_blank">AWS WAF</a> logs</li> 
</ul> 
<h2>Deploy your application</h2> 
<p>Follow the instructions on the <a href="https://github.com/aws-samples/aws-three-tier-web-architecture-workshop" rel="noopener" target="_blank">GitHub repo</a> and instructions in the <a href="https://catalog.us-east-1.prod.workshops.aws/workshops/85cd2bb2-7f79-4e96-bdee-8078e469752a/en-US" rel="noopener" target="_blank">AWS Three Tier Web Architecture workshop</a> to deploy your application.</p> 
<p>After you install the app, get your credentials from AWS to use with Elastic’s AWS integration.</p> 
<p>There are several options for credentials:</p> 
<ul> 
 <li>Use access keys directly</li> 
 <li>Use temporary security credentials</li> 
 <li>Use a shared credentials file</li> 
 <li>Use an <a href="http://aws.amazon.com/iam" rel="noopener" target="_blank">AWS Identity and Access Management</a> (IAM) role Amazon Resource Name (ARN)</li> 
</ul> 
<p>For more details, refer to <a href="https://docs.elastic.co/en/integrations/aws#aws-credentials" rel="noopener" target="_blank">AWS Credentials</a> and <a href="https://docs.elastic.co/en/integrations/aws#aws-permissions" rel="noopener" target="_blank">AWS Permissions</a>.</p> 
<h2>Configure VPC flow logs to send to Kinesis Data Firehose</h2> 
<p>In the VPC for the application you deployed, you need to configure your VPC flow logs and point them to the Firehose delivery stream.</p> 
<p><img alt="" class="alignnone size-full wp-image-44527" height="204" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/03/13/bdb3086-elastic-kinesis-firehose-7.jpg" style="border: 1px solid #cccccc; margin: 10px 0px 10px 0px;" width="600" /></p> 
<h2>Validate the VPC flow logs</h2> 
<p>In the Elastic Observability view of the log streams, you should see the VPC flow logs coming in after a few minutes, as shown in the following screenshot.</p> 
<p><img alt="" class="alignnone size-full wp-image-44528" height="796" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/03/13/bdb3086-elastic-kinesis-firehose-8.jpg" style="border: 1px solid #cccccc; margin: 10px 0px 10px 0px;" width="1600" /></p> 
<h2>Analyze VPC flow logs in Elastic</h2> 
<p>Now that you have VPC flow logs in Elastic Cloud, how can you analyze them? There are several analyses you can perform on the VPC flow log data:</p> 
<ul> 
 <li>Use Elastic’s Analytics Discover capabilities to manually analyze the data</li> 
 <li>Use Elastic Observability’s anomaly feature to identify anomalies in the logs</li> 
 <li>Use an out-of-the-box dashboard to further analyze the data</li> 
</ul> 
<h3>Use Elastic’s Analytics Discover to manually analyze data</h3> 
<p>In Elastic Analytics, you can search and filter your data, get information about the structure of the fields, and display your findings in a visualization. You can also customize and save your searches and place them on a dashboard.</p> 
<p>For a complete understanding of Discover and all of Elastic’s Analytics capabilities, refer to <a href="https://www.elastic.co/guide/en/kibana/current/discover.html" rel="noopener" target="_blank">Discover</a>.</p> 
<p>For VPC flow logs, it’s important to understand the following:</p> 
<ul> 
 <li>How many logs were accepted or rejected</li> 
 <li>Where potential security violations occur (source IPs from outside the VPC)</li> 
 <li>What port is generally being queried</li> 
</ul> 
<p>For our example, we filter the logs on the following:</p> 
<ul> 
 <li><strong>Delivery stream name</strong> – <code>AWS-3-TIER-APP-VPC-LOGS</code></li> 
 <li><strong>VPC flow log action</strong> – <code>REJECT</code></li> 
 <li><strong>Time frame</strong> – 5 hours</li> 
 <li><strong>VPC network interface</strong> – Webserver 1 and Webserver 2 interfaces</li> 
</ul> 
<p>We want to see what IP addresses are trying to hit our web servers. From that, we want to understand which IP addresses we’re getting the most <code>REJECT</code> actions from. We simply find the <code>source.ip</code> field and can quickly get a breakdown that shows <code>185.156.73.54</code> is the most rejected for the last 3 or more hours we’ve turned on VPC flow logs.</p> 
<p><img alt="" class="alignnone size-full wp-image-44529" height="841" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/03/13/bdb3086-elastic-kinesis-firehose-9.jpg" style="border: 1px solid #cccccc; margin: 10px 0px 10px 0px;" width="1600" /></p> 
<p>Additionally, we can create a visualization by choosing <strong>Visualize</strong>. We get the following donut chart, which we can add to a dashboard.</p> 
<p><img alt="" class="alignnone size-full wp-image-44530" height="937" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/03/13/bdb3086-elastic-kinesis-firehose-10.jpg" style="border: 1px solid #cccccc; margin: 10px 0px 10px 0px;" width="1600" /></p> 
<p>Additionally to IP addresses, we want to also see what port is being hit on our web servers.</p> 
<p>We select the destination port field, and the pop-up shows us that port <code>8081</code> is being targeted. This port is generally used for the administration of Apache Tomcat. This is a potential security issue, however port 8081 is turned off for outside traffic, hence the <code>REJECT</code>.</p> 
<p><img alt="" class="alignnone size-full wp-image-44531" height="848" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/03/13/bdb3086-elastic-kinesis-firehose-11.jpg" style="border: 1px solid #cccccc; margin: 10px 0px 10px 0px;" width="800" /></p> 
<h3>Detect anomalies in Elastic Observability logs</h3> 
<p>In addition to Discover, Elastic Observability provides the ability to detect anomalies on logs using machine learning (ML). The feature has the following options:</p> 
<ul> 
 <li><strong>Log rate</strong> – Automatically detects anomalous log entry rates</li> 
 <li><strong>Categorization</strong> – Automatically categorizes log messages</li> 
</ul> 
<p><img alt="" class="alignnone size-full wp-image-44532" height="631" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/03/13/bdb3086-elastic-kinesis-firehose-12.jpg" style="border: 1px solid #cccccc; margin: 10px 0px 10px 0px;" width="1600" /></p> 
<p>For our VPC flow log, we enabled both features. When we look at what was detected for anomalous log entry rates, we get the following results.</p> 
<p><img alt="" class="alignnone size-full wp-image-44533" height="791" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/03/13/bdb3086-elastic-kinesis-firehose-13.jpg" style="border: 1px solid #cccccc; margin: 10px 0px 10px 0px;" width="1600" /></p> 
<p>Elastic immediately detected a spike in logs when we turned on VPC flow logs for our application. The rate change is being detected because we’re also ingesting VPC flow logs from another application for a couple of days prior to adding the application in this post.</p> 
<p>We can drill down into this anomaly with ML and analyze further.</p> 
<p><img alt="" class="alignnone size-full wp-image-44534" height="983" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/03/13/bdb3086-elastic-kinesis-firehose-14.jpg" style="border: 1px solid #cccccc; margin: 10px 0px 10px 0px;" width="1600" /></p> 
<p>To learn more about the ML analysis you can utilize with your logs, refer to <a href="https://www.elastic.co/guide/en/kibana/8.5/xpack-ml.html" rel="noopener" target="_blank">Machine learning</a>.</p> 
<p>Because we know that a spike exists, we can also use the Elastic AIOps Labs Explain Log Rate Spikes capability. Additionally, we’ve grouped them to see what is causing some of the spikes.</p> 
<p><img alt="" class="alignnone size-full wp-image-44535" height="803" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/03/13/bdb3086-elastic-kinesis-firehose-15.jpg" style="border: 1px solid #cccccc; margin: 10px 0px 10px 0px;" width="1600" /></p> 
<p>In the preceding screenshot, we can observe that a specific network interface is sending more VPC log flows than others. We can drill down into this further in Discover.</p> 
<h3>Use the VPC flow log dashboard</h3> 
<p>Finally, Elastic also provides an out-of-the-box dashboard to show the top IP addresses hitting your VPC, geographically where they are coming from, the time series of the flows, and a summary of VPC flow log rejects within the time frame.</p> 
<p>You can enhance this baseline dashboard with the visualizations you find in Discover, as we discussed earlier.</p> 
<p><img alt="" class="alignnone size-full wp-image-44536" height="1041" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/03/13/bdb3086-elastic-kinesis-firehose-16.jpg" style="border: 1px solid #cccccc; margin: 10px 0px 10px 0px;" width="1600" /></p> 
<h2>Conclusion</h2> 
<p>This post demonstrated how to configure an integration with Kinesis Data Firehose and Elastic for efficient infrastructure monitoring of VPC flow logs in Elastic Kibana dashboards. Elastic offers flexible deployment options on AWS, supporting software as a service (SaaS), <a href="https://aws.amazon.com/marketplace/pp/Elasticsearch-Inc-Elasticsearch-Service-on-Elastic/B01N6YCISK" rel="noopener" target="_blank">AWS Marketplace</a>, and bring your own license (BYOL) deployments. Elastic also provides AWS Marketplace private offers. You have the option to deploy and run the Elastic Stack yourself within your AWS account, either free or with a paid subscription from <a href="https://www.elastic.co/blog/getting-started-with-elastic-cloud-on-amazon-web-services-aws" rel="noopener" target="_blank">Elastic</a>. To get started, visit the Kinesis Data Firehose console and specify Elastic as the destination. To learn more, explore the <a href="https://docs.aws.amazon.com/firehose/latest/dev/what-is-this-service.html" rel="noopener" target="_blank">Amazon Kinesis Data Firehose Developer Guide</a>.</p> 
<hr /> 
<h3>About the Authors</h3> 
<p style="clear: both;"><strong><img alt="" class="alignleft size-full wp-image-44556" height="127" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/03/14/udayasimha.jpg" width="100" />Udayasimha Theepireddy </strong>is an Elastic Principal Solution Architect, where he works with customers to solve real world technology problems using Elastic and AWS services. He has a strong background in technology, business, and analytics.</p> 
<p style="clear: both;"><img alt="" class="size-full wp-image-44543 alignleft" height="109" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/03/13/antony-prasad-thevaraj-100.jpg" width="100" /><strong>Antony Prasad Thevaraj</strong> is a Sr. Partner Solutions Architect in Data and Analytics at AWS. He has over 12 years of experience as a Big Data Engineer, and has worked on building complex ETL and ELT pipelines for various business units.</p> 
<p style="clear: both;"><strong><img alt="" class="alignleft size-full wp-image-44548" height="100" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/03/14/Mostafa-Mansour-100.jpg" width="100" />Mostafa Mansour</strong> is a Principal Product Manager – Tech at Amazon Web Services where he works on Amazon Kinesis Data Firehose. He specializes in developing intuitive product experiences that solve complex challenges for customers at scale. When he’s not hard at work on Amazon Kinesis Data Firehose, you’ll likely find Mostafa on the squash court, where he loves to take on challengers and perfect his dropshots.</p>
