---
title: "Amazon Kinesis Data Streams launches On-demand Advantage for instant throughput increases and streaming at scale"
url: "https://aws.amazon.com/blogs/big-data/amazon-kinesis-data-streams-launches-on-demand-advantage-for-instant-throughput-increases-and-streaming-at-scale/"
date: "Mon, 03 Nov 2025 22:00:31 +0000"
author: "Pratik Patel"
feed_url: "https://aws.amazon.com/blogs/big-data/category/analytics/amazon-kinesis/feed/"
---
<p>Today, AWS announced the new <a href="https://aws.amazon.com/kinesis/data-streams/" rel="noopener noreferrer" target="_blank">Amazon Kinesis Data Streams</a> On-demand Advantage mode, which includes warm throughput capability and an updated pricing structure. With this feature you can enable instant scaling for traffic surges while optimizing costs for consistent streaming workloads. <a href="https://docs.aws.amazon.com/streams/latest/dev/how-do-i-size-a-stream.html#ondemandmode" rel="noopener noreferrer" target="_blank">On-demand Advantage</a> mode is a cost-effective way to stream with Kinesis Data Streams for use cases that ingest at least 10 MiB/s in aggregate or have hundreds of data streams in an AWS Region.</p> 
<p>In this post, we explore this new feature, including key use cases, configuration options, pricing considerations, and best practices for optimal performance.</p> 
<h2>Real-world use cases</h2> 
<p>As streaming data volumes grow and use cases evolve, you can face two common challenges with your streaming workloads:</p> 
<p><strong>Challenge 1: Preparing for traffic spikes</strong></p> 
<p>Many businesses experience predictable but significant traffic surges during events like product launches, content releases, or holiday sales. Using an on-demand capacity mode, you have to complete several steps when preparing for traffic spikes:</p> 
<ul> 
 <li>Transition to provisioned mode</li> 
 <li>Manually estimate and increase shards based on anticipated peak demand</li> 
 <li>Wait for scaling operations to finish</li> 
 <li>Subsequently return to on-demand mode</li> 
</ul> 
<p>This mode-switching process was time consuming, required careful planning, and introduced operational complexity, forcing customers to either accept this operational burden, overprovision capacity well in advance, or risk throttling during critical business periods when data ingestion reliability matters most.</p> 
<p><strong>Challenge 2: Cost optimization for consistent workloads</strong></p> 
<p>Organizations with large, consistent streaming workloads want to optimize costs without sacrificing the simplicity and scalability available with on-demand streams. On-demand capacity mode serves well for fluctuating data traffic, yet customers desired a more economical approach to handle high-volume streaming workloads.</p> 
<p>On-demand Advantage directly address both challenges by providing the capability to warm on-demand streams and a new pricing structure. With the new On-demand Advantage mode, there is no longer a fixed, per-stream charge, and the throughput usage is priced at a lower rate. The only requirement is that the account commits to streaming with at least <a href="https://aws.amazon.com/kinesis/data-streams/pricing/" rel="noopener noreferrer" target="_blank">25 MiB/s of data ingest and 25 MiB/s of data retrieval</a> usage.</p> 
<p>This launch improves data streaming across multiple industries:</p> 
<ul> 
 <li>Online gaming companies can now prepare their streams for game launches without the cumbersome process of switching between modes and manually calculating shard requirements</li> 
 <li>Media and entertainment providers can support smooth data ingestion during major content releases and live events</li> 
 <li>E-commerce services can handle holiday sales traffic while optimizing costs for their baseline workloads.</li> 
</ul> 
<p>By combining instant scaling with cost efficiency, you can confidently manage both predictable traffic surges and consistent streaming volumes without compromising on performance or budget.</p> 
<h2>How it works</h2> 
<p>The key features of On-demand Advantage mode are warm throughput and committed-usage pricing.</p> 
<h3>Warm throughput</h3> 
<p>With the warm throughput feature, available once you’ve enabled On-demand Advantage mode, you can configure your Kinesis Data Streams on-demand streams to have instantly available throughput capacity up to 10 GiB/s. This means you can proactively prepare on-demand streams for expected peak traffic events without the cumbersome process of switching between provisioned modes and manually calculating shard requirements. Key benefits include:</p> 
<ul> 
 <li>The ability to prepare for peak events so you can handle traffic surges smoothly</li> 
 <li>Alleviation of the need to build custom scaling solutions</li> 
 <li>The capability to continue scaling automatically beyond warm throughput if needed, up to 10 GiB/s or 10 million events per second</li> 
 <li>No additional fee for maintaining warm capacity</li> 
</ul> 
<h3>Committed-usage pricing</h3> 
<p>When you’ve enabled On-demand Advantage mode, the billing for the on-demand streams switches to a new structure that removes the stream hour charge and offers <a href="https://aws.amazon.com/kinesis/data-streams/pricing/" rel="noopener noreferrer" target="_blank">a discount of at least 60%</a> for the throughput usage. Based on US East (N. Virginia) pricing, data ingested is priced 60% lower, data retrieval is priced 60% lower, Enhanced fan-out data retrieval is 68% lower, and extended retention is priced 77% lower. In return, you commit to stream 25 MiB/s for at least 24 hours. Even when actual usage is lower, if you enable this setting, you’re charged for the minimum 25 MiB/s throughput at the discounted price. Overall, the signficant discounts offered means that On-demand Advantage is more cost-effective for use cases that ingest at least 10 MiB/s in aggregate, fan out to more than two consumer applications, or have hundreds of data streams in an AWS Region.</p> 
<h2>Getting started</h2> 
<p>Follow these steps to start using On-demand Advantage mode.</p> 
<h3>Enabling On-demand Advantage mode</h3> 
<p>To start using the On-demand Advantage mode:</p> 
<p><strong>In the AWS Management Console </strong></p> 
<ol> 
 <li>Navigate to the Kinesis Data Streams console</li> 
 <li>Navigate to the <strong>Account Settings</strong> tab</li> 
 <li>Choose <strong>Edit billing mode</strong></li> 
 <li>Select the <strong>On-demand Advantage</strong> option</li> 
 <li>Select the checkbox, <strong>I acknowledge this change cannot be reverted for 24 hours</strong></li> 
 <li>Choose <strong>Save changes</strong></li> 
</ol> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/10/31/BDB-5596-od-billing-mode.png"><img alt="on-demand-billing-mode" class="alignnone size-full wp-image-84904" height="429" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/10/31/BDB-5596-od-billing-mode.png" style="margin: 10px 0px 10px 0px;" width="1422" /></a></p> 
<p><strong>Using the AWS CLI</strong></p> 
<p>You can run the following CLI command to enable the minimum throughput billing commitment:</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">aws kinesis update-account-settings \
--minimum-throughput-billing-commitment Status=ENABLED</code></pre> 
</div> 
<p><strong>Using the AWS SDK</strong></p> 
<p>You can use the SDK to enable the minimum throughput billing commitment. The following Python example shows how to do it:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">import boto3

client = boto3.client('kinesis')
response = client.update_account_settings(
    MinimumThroughputBillingCommitment={"Status": "ENABLED"}
)</code></pre> 
</div> 
<p>Once enabled, you commit your stream to this pricing mode for a minimum period of 24 hours, after which you can opt out as needed.</p> 
<h3>Configuring warm throughput</h3> 
<p>To start using warm throughput for Kinesis Data Streams On-demand:</p> 
<p><strong>Using the AWS Management Console </strong></p> 
<ol> 
 <li>Navigate to the Kinesis Data Streams console</li> 
 <li>Select your stream and go to the Configuration tab</li> 
 <li>Choose <strong>Edit</strong> next to Warm Throughput</li> 
 <li>Set your desired warm throughput (up to 10 GiB/s)</li> 
 <li>Save your changes</li> 
</ol> 
<p><strong>Using the AWS CLI</strong></p> 
<p>You can run the following CLI command to enable the warm throughput:</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">aws kinesis update-stream-warm-throughput \
  --stream-name MyStream \
  --warm-throughput-mi-bps 1000</code></pre> 
</div> 
<p><strong>Using the AWS SDK:</strong></p> 
<p>You can use the SDK to enable warm throughput. The following Python example shows how to do it:</p> 
<div class="hide-language"> 
 <pre><code class="lang-python">import boto3

client = boto3.client('kinesis')
response = client.update_stream_warm_throughput(
    StreamName='MyStream',
    WarmThroughputMiBps=1000
)</code></pre> 
</div> 
<p>You can also create a new on-demand stream with warm throughput using the existing <a href="https://docs.aws.amazon.com/kinesis/latest/APIReference/API_CreateStream.html" rel="noopener noreferrer" target="_blank">CreateStream</a> API, or set warm throughput when converting a data stream from provisioned to On-demand Advantage mode.</p> 
<h2>Throttling and best practices for optimal performance</h2> 
<p>When working with warm throughput, it’s important to understand how capacity is managed. Each stream can instantly handle traffic up to the configured warm throughput level and will automatically scale beyond that as needed.</p> 
<p>For optimal performance with warm throughput:</p> 
<ol> 
 <li><strong>Use a uniformly distributed partition key strategy</strong> to evenly distribute records across shards and avoid hotspots and consider your partition key strategy carefully as you can ingest a maximum of 1 MiB/s of data per partition key, regardless of the warm throughput configured.</li> 
 <li><a href="https://docs.aws.amazon.com/streams/latest/dev/monitoring-with-cloudwatch.html#cloudwatch-metrics" rel="noopener noreferrer" target="_blank"><strong>Monitor throughput metrics</strong></a> to adjust warm throughput settings based on actual usage patterns.</li> 
 <li><a href="https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/retry-backoff.html" rel="noopener noreferrer" target="_blank"><strong>Implement backoff and retry logic</strong></a> in producer applications to handle potential throttling.</li> 
</ol> 
<p>For cost optimization with committed usage pricing:</p> 
<ol> 
 <li><strong>Analyze your daily throughput</strong> to verify it is at least 10 MiB/s.</li> 
 <li><strong>Consider consolidating streams</strong> across your organization to maximize the benefit of the discount for on-demand streams.</li> 
 <li><strong>Use cost effective data retrievals with – Use Enhanced Fan-Out </strong>– Use Enhanced Fan-Out consumers for applications that need dedicated throughput with <a href="https://aws.amazon.com/kinesis/data-streams/pricing/" rel="noopener noreferrer" target="_blank">68% lower data retrievals cost</a> in advantage mode.</li> 
</ol> 
<h2>Warm throughput in action</h2> 
<p>To demonstrate how warm throughput behaves, we enabled committed pricing in an AWS account and created two on-demand streams: “KDS-OD-STANDARD” and “KDS-OD-WARM-TP”. The “KDS-OD-WARM-TP” stream was configured with 100 MiB/second warm throughput, while “KDS-OD-STANDARD” remained as a regular on-demand stream without warm throughput, as demonstrated in the following screenshot.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/10/31/BDB-5596-streams.png"><img alt="od-standard-warm-streams" class="alignnone size-full wp-image-84905" height="240" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/10/31/BDB-5596-streams.png" style="margin: 10px 0px 10px 0px;" width="1378" /></a></p> 
<p>In our experiment, we initially simulated approximately 2 MiB/second traffic ingest for both “KDS-OD-STANDARD” and “KDS-OD-WARM-TP” streams. We used a UUID as a partition key so that traffic was evenly distributed across the shards of the Kinesis data streams, helping prevent potential hotspots that might skew our results. After establishing this baseline, we increased the ingest traffic to around 28 MiB/second within 10 minutes. We then further escalated the traffic to exceed 60 MiB/second within 15 minutes of the initial increase, as illustrated in the following screenshot.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/10/31/BDB-5596-streams-ingest-throughput.png"><img alt="streams-ingest-mb-second-metric" class="alignnone size-full wp-image-84906" height="386" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/10/31/BDB-5596-streams-ingest-throughput.png" style="margin: 10px 0px 10px 0px;" width="1379" /></a></p> 
<p>The following graph shows the <code>ThrottledRecords</code> CloudWatch metric for both “KDS-OD-STANDARD” and “KDS-OD-WARM-TP” that the warm throughput-enabled stream (“KDS-OD-WARM-TP”) did not encounter throttles during both traffic spikes, as it had 100 MiB/second warm throughput configured. In contrast, the standard on-demand stream (“KDS-OD-STANDARD”) experienced throttling when we increased traffic by 14x initially and by 2x later, before eventually scaling to bring throttles back to zero. This experiment demonstrates that you can use warm throughput to instantly prepare for peak usage times and avoid throttling during sudden traffic increases.</p> 
<p><a href="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/10/31/BDB-5596-streams-throttle-cw-metrics.png"><img alt="streams-throttle-metrics" class="alignnone size-full wp-image-84907" height="397" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/10/31/BDB-5596-streams-throttle-cw-metrics.png" style="margin: 10px 0px 10px 0px;" width="1380" /></a></p> 
<h2>Conclusion</h2> 
<p>As we outlined in this post, the new Amazon Kinesis Data Streams On-demand Advantage mode provides significant benefits for organizations of different sizes:</p> 
<ul> 
 <li><strong>Instant scaling</strong> for predictable traffic surges without overprovisioning.</li> 
 <li><strong>Cost optimization</strong> for consistent streaming workloads with at least <a href="https://aws.amazon.com/kinesis/data-streams/pricing/" rel="noopener noreferrer" target="_blank">60% discount</a>.</li> 
 <li><strong>Simplified operations</strong> with no need to switch between different capacity modes.</li> 
 <li><strong>Enhanced flexibility</strong> to handle both expected and unexpected traffic patterns.</li> 
</ul> 
<p>With these enhancements you can build and operate real-time streaming applications at many scales. Kinesis Data Streams now provides the ideal combination of scalability, performance, and cost-efficiency.</p> 
<p>To learn more about these new features, visit the <a href="https://aws.amazon.com/kinesis/data-streams" rel="noopener noreferrer" target="_blank">Amazon Kinesis Data Streams documentation</a>.</p> 
<hr /> 
<h3>About the authors</h3> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Roy (KDS) Wang" class="aligncenter size-full wp-image-29797" height="160" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/10/31/BDB-5596-author-3.png" width="120" />
  </div> 
  <h3 class="lb-h4">Roy (KDS) Wang</h3> 
  <p><a href="https://www.linkedin.com/in/ruotianwang/" rel="noopener" target="_blank">Roy</a> is a Senior Product Manager with Amazon Kinesis Data Streams. He is passionate about learning from and collaborating with customers to help organizations run faster and smarter. Outside of work, Roy strives to be a good dad to his new son and builds plastic model kits.</p>
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Pratik Patel" class="aligncenter size-full wp-image-29797" height="160" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/10/31/BDB-5596-author-1.jpeg" width="120" />
  </div> 
  <h3 class="lb-h4">Pratik Patel</h3> 
  <p><a href="https://www.linkedin.com/in/pratikpatel-aws/" rel="noopener" target="_blank">Pratik</a> is Sr. Technical Account Manager and streaming analytics specialist. He works with AWS customers and provides ongoing support and technical guidance to help plan and build solutions using best practices and proactively keep customers’ AWS environments operationally healthy.</p>
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Umesh Chaudhari" class="aligncenter size-full wp-image-29797" height="160" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/10/31/BDB-5596-author-2.jpeg" width="120" />
  </div> 
  <h3 class="lb-h4">Umesh Chaudhari</h3> 
  <p><a href="https://www.linkedin.com/in/utc/" rel="noopener" target="_blank">Umesh</a> is a Sr. Streaming Solutions Architect at AWS. He works with customers to design and build real-time data processing systems. He has extensive working experience in software engineering, including architecting, designing, and developing data analytics systems. Outside of work, he enjoys traveling, following tech trends.</p>
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Simon Peyer" class="aligncenter size-full wp-image-29797" height="160" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/10/31/BDB-5596-author-4.jpeg" width="120" />
  </div> 
  <h3 class="lb-h4">Simon Peyer</h3> 
  <p><a href="https://www.linkedin.com/in/peyers/" rel="noopener" target="_blank">Simon</a> is a Solutions Architect at AWS based in Switzerland. He is a practical doer and passionate about connecting technology and people using AWS Cloud services. A special focus for him is data streaming and automations. Besides work, Simon enjoys his family, the outdoors, and hiking in the mountains.20</p>
 </div> 
</footer>
