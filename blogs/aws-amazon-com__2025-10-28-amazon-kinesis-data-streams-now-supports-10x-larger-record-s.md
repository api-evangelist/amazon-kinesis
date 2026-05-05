---
title: "Amazon Kinesis Data Streams now supports 10x larger record sizes: Simplifying real-time data processing"
url: "https://aws.amazon.com/blogs/big-data/amazon-kinesis-data-streams-now-supports-10x-larger-record-sizes-simplifying-real-time-data-processing/"
date: "Tue, 28 Oct 2025 19:23:30 +0000"
author: "Sumant Nemmani"
feed_url: "https://aws.amazon.com/blogs/big-data/category/analytics/amazon-kinesis/feed/"
---
<p>Today, AWS announced that <a href="https://aws.amazon.com/kinesis/data-streams/" rel="noopener" target="_blank">Amazon Kinesis Data Streams</a> now supports record sizes up to 10MiB – a tenfold increase from the previous limit. With this launch, you can now publish intermittent larger data payloads on your data streams while continuing to use existing Kinesis Data Streams APIs in your applications without additional effort. This launch is accompanied by a 2x increase in the maximum <code>PutRecords</code> request size from 5MiB to 10MiB, simplifying data pipelines and reducing operational overhead for IoT analytics, change data capture, and generative AI workloads.</p> 
<p>In this post, we explore Amazon Kinesis Data Streams large record support, including key use cases, configuration of maximum record sizes, throttling considerations, and best practices for optimal performance.</p> 
<h2>Real world use cases</h2> 
<p>As data volumes grow and use cases evolve, we’ve seen increasing demand for supporting larger record sizes in streaming workloads. Previously, when you needed to process records larger than 1MiB, you had two options:</p> 
<ul> 
 <li>Split large records into multiple smaller records in producer applications and reassemble them in consumer applications</li> 
 <li>Store large records in <a href="https://aws.amazon.com/s3/" rel="noopener noreferrer" target="_blank">Amazon Simple Storage Service (Amazon S3</a>) and send only metadata through Kinesis Data Streams</li> 
</ul> 
<p>Both these approaches are useful, but they add complexity to data pipelines, requiring additional code, increasing operational overhead, and complicating error handling and debugging, particularly when customers need to stream large records intermittently.</p> 
<p>This enhancement improves the ease of use and reduces operational overhead for customers handling intermittent data payloads across various industries and use cases. In the IoT analytics domain, connected vehicles and industrial equipment are generating increasing volumes of sensor telemetry data, with the size of individual telemetry records occasionally exceeding the previous 1MiB limit in Kinesis. This required customers to implement complex workarounds, such as splitting large records into multiple smaller ones or storing the large records separately and only sending metadata through Kinesis. Similarly, in database change data capture (CDC) pipelines, large transaction records can be produced, especially during bulk operations or schema changes. In the machine learning and generative AI space, workflows are increasingly requiring the ingestion of larger payloads to support richer feature sets and multi-modal data types like audio and images. The increased Kinesis record size limit from 1MiB to 10MiB limits the need for these types of complex workarounds, simplifying data pipelines and reducing operational overhead for customers in IoT, CDC, and advanced analytics use cases. Customers can now more easily ingest and process these intermittent large data records using the same familiar Kinesis APIs.</p> 
<h2>How it works</h2> 
<p>To start processing larger records:</p> 
<ol> 
 <li>Update your stream’s maximum record size limit (<code>maxRecordSize</code>) through the AWS Console, AWS CLI, or AWS SDKs.</li> 
 <li>Continue using the same <code>PutRecord</code> and <code>PutRecords</code> APIs for producers.</li> 
 <li>Continue using the same <code>GetRecords</code> or <code>SubscribeToShard</code> APIs for consumers.</li> 
</ol> 
<p>Your stream will be in <code>Updating</code> status for a few seconds before being ready to ingest larger records.</p> 
<h2>Getting started</h2> 
<p>To start processing larger records with Kinesis Data Streams, you can update the maximum record size by using the AWS Management Console, CLI or SDK.</p> 
<p>On the AWS Management Console, </p> 
<ol> 
 <li>Navigate to the Kinesis Data Streams console.</li> 
 <li>Choose your stream and select the <strong>Configuration</strong> tab.</li> 
 <li>Choose <strong>Edit</strong> (next to <strong>Maximum record size</strong>).</li> 
 <li>Set your desired maximum record size (up to 10MiB).</li> 
 <li>Save your changes.</li> 
</ol> 
<p><strong>Note:</strong> This setting only adjusts the maximum record size for this Kinesis data stream. Before increasing this limit, verify that all downstream applications can handle larger records.</p> 
<p>Most common consumers such as Kinesis Client Library (starting with version 2.x), <a href="https://aws.amazon.com/firehose/" rel="noopener" target="_blank">Amazon Data Firehose</a> delivery to <a href="https://aws.amazon.com/s3/" rel="noopener" target="_blank">Amazon S3</a> and <a href="https://aws.amazon.com/lambda/" rel="noopener" target="_blank">AWS Lambda</a> support processing records larger than 1 MiB. To learn more, refer to the Amazon Kinesis Data Streams <a href="https://docs.aws.amazon.com/streams/latest/dev/large-records.html" rel="noopener noreferrer" target="_blank">documentation for large records</a>.</p> 
<p>You can also update this setting using the AWS CLI: </p> 
<div class="hide-language"> 
 <pre><code class="lang-code">aws kinesis update-max-record-size \
--stream-arn &lt;stream-arn&gt; \
--max-record-size-in-ki-b 5000</code></pre> 
</div> 
<p>Or using the AWS SDK:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">import boto3

client = boto3.client('kinesis')
response = client.update_max_record_size(
StreamARN='arn:aws:kinesis:us-west-2:123456789012:stream/my-stream',
MaxRecordSizeInKiB=5000
)</code></pre> 
</div> 
<p><strong>Throttling and best practices for optimal performance</strong></p> 
<p>Individual shard throughput limits of 1MiB/s for writes and 2MiB/s for reads remain unchanged with support for larger record sizes. To work with large records, let’s understand how throttling works. In a stream, each shard has a throughput capacity of 1 MiB per second. To accommodate large records, each shard temporarily bursts up to 10MiB/s, eventually averaging out to 1MiB per second. To help visualize this behavior, think of each shard having a <em>capacity tank</em> that refills at 1MiB per second. After sending a large record (for example, a 10MiB record), the tank begins refilling immediately, allowing you to send smaller records as capacity becomes available. This capacity to support large records is continuously refilled into the stream. The rate of refilling depends on the size of the large records, the size of the baseline record, the overall traffic pattern, and your chosen partition key strategy. When you process large records, each shard continues to process baseline traffic while leveraging its burst capacity to handle these larger payloads.</p> 
<p>To illustrate how Kinesis Data Streams handles different proportions of large records, let’s examine the results a simple test. For our test configuration, we set up a producer that sends data to an on-demand stream (defaults to 4 shards) at a rate of 50 records per second. The baseline records are 10KiB in size, while large records are 2MiB each. We conducted multiple test cases by progressively increasing the proportion of large records from 1% to 5% of the total stream traffic, along with a baseline case containing no large records. To ensure consistent testing conditions, we distributed the large records uniformly over time for example, in the 1% scenario, we sent one large record for every 100 baseline records. The following graph shows the results: </p> 
<p><img src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/10/16/BDB-5568-1.png" /></p> 
<p>In the graph, horizontal annotations indicate throttling occurrence peaks. The baseline scenario, represented by the blue line, shows minimal throttling events. As the proportion of large records increases from 1% to 5%, we observe an increase in the rate at which your stream throttles your data, with a notable acceleration in throttling events between the 2% and 5% scenarios. This test demonstrates how Kinesis Data Streams manages increasing proportion of large records. </p> 
<p>We recommend maintaining large records at 1-2% of your total record count for optimal performance. In production environments, actual stream behavior varies based on three key factors: the size of baseline records, the size of large records, and the frequency at which large records appear in the stream. We recommend that you test with your demand pattern to determine the specific behavior.</p> 
<p>With on-demand streams, when the incoming traffic exceeds 500 KB/s per shard, it splits the shard within 15 minutes. The parent shard’s hash key values are redistributed evenly across child shards. Kinesis automatically scales the stream to increase the number of shards, enabling distribution of large records across a larger number of shards depending on the partition key strategy employed.</p> 
<p>For optimal performance with large records:</p> 
<ol> 
 <li>Use a random partition key strategy to distribute large records evenly across shards.</li> 
 <li>Implement backoff and retry logic in producer applications.</li> 
 <li>Monitor shard-level metrics to identify potential bottlenecks.</li> 
</ol> 
<p>If you still need to continuously stream of large records, consider using Amazon S3 to store payloads and send only metadata references to the stream. Refer to <a href="http://aws.amazon.com/blogs/big-data/processing-large-records-with-amazon-kinesis-data-streams/" rel="noopener noreferrer" target="_blank">Processing large records with Amazon Kinesis Data Streams</a> for more information.</p> 
<p><strong>Conclusion </strong></p> 
<p>Amazon Kinesis Data Streams now supports record sizes up to 10MiB, a tenfold increase from the previous 1MiB limit. This enhancement simplifies data pipelines for IoT analytics, change data capture, and AI/ML workloads by eliminating the need for complex workarounds. You can continue using existing Kinesis Data Streams APIs without additional code changes and benefit from increased flexibility in handling intermittent large payloads.</p> 
<ul> 
 <li>For optimal performance, we recommend maintaining large records at 1-2% of total record count.</li> 
 <li>For best results with large records, implement a uniformly distributed partition key strategy to evenly distribute records across shards, include backoff and retry logic in producer applications, and monitor shard-level metrics to identify potential bottlenecks.</li> 
 <li>Before increasing the maximum record size, verify that all downstream applications and consumers can handle larger records.</li> 
</ul> 
<p>We’re excited to see how you’ll leverage this capability to build more powerful and efficient streaming applications. To learn more, visit the <a href="https://docs.aws.amazon.com/streams/latest/dev/large-records.html" rel="noopener noreferrer" target="_blank">Amazon Kinesis Data Streams documentation</a>.</p> 
<hr /> 
<h3>About the authors</h3> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Sumant Nemmani" class="aligncenter size-full wp-image-29797" height="160" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/10/16/BDB-5568-2.jpg" width="120" />
  </div> 
  <h3 class="lb-h4">Sumant Nemmani</h3> 
  <p><a href="https://www.linkedin.com/in/sumantnemmani/" rel="noopener" target="_blank">Sumant</a> is a product manager for Amazon Kinesis Data Streams. He is passionate about learning from customers and enjoys helping them unlock value with AWS. Outside of work, he spends time making music with his band Project Mishram, exploring history and food while traveling, and long-form podcasts on technology and history.</p>
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Umesh Chaudhari" class="aligncenter size-full wp-image-29797" height="160" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/10/16/BDB-5568-3.jpeg" width="120" />
  </div> 
  <h3 class="lb-h4">Umesh Chaudhari</h3> 
  <p><a href="https://www.linkedin.com/in/utc/" rel="noopener" target="_blank">Umesh</a> is a Sr. Streaming Solutions Architect at AWS. He works with customers to design and build real-time data processing systems. He has extensive working experience in software engineering, including architecting, designing, and developing data analytics systems. Outside of work, he enjoys traveling, following tech trends</p>
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Pratik Patel" class="aligncenter size-full wp-image-29797" height="160" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/10/16/BDB-5568-4.jpeg" width="120" />
  </div> 
  <h3 class="lb-h4">Pratik Patel</h3> 
  <p><a href="https://www.linkedin.com/in/pratikpatel-aws/" rel="noopener" target="_blank">Pratik</a> is Sr. Technical Account Manager and streaming analytics specialist. He works with AWS customers and provides ongoing support and technical guidance to help plan and build solutions using best practices and proactively keep customers’ AWS environments operationally healthy.</p>
 </div> 
</footer>
