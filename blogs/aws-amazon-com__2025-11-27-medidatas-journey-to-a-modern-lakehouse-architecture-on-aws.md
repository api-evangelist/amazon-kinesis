---
title: "Medidata’s journey to a modern lakehouse architecture on AWS"
url: "https://aws.amazon.com/blogs/big-data/medidatas-journey-to-a-modern-lakehouse-architecture-on-aws/"
date: "Thu, 27 Nov 2025 01:00:46 +0000"
author: "Mike Araujo"
feed_url: "https://aws.amazon.com/blogs/big-data/category/analytics/amazon-kinesis/feed/"
---
<p><em>This post was co-authored by Mike Araujo Principal Engineer at Medidata Solutions.</em></p> 
<p>The life sciences industry is transitioning from fragmented, standalone tools towards integrated, platform-based solutions. <a href="https://www.medidata.com/en/" rel="noopener noreferrer" target="_blank">Medidata</a>, a Dassault Systèmes company, is building a next-generation data platform that addresses the complex challenges of modern clinical research. In this post, we show you how Medidata created a unified, scalable, real-time data platform that serves thousands of clinical trials worldwide with AWS services, <a href="https://iceberg.apache.org/" rel="noopener noreferrer" target="_blank">Apache Iceberg</a>, and a modern lakehouse architecture.</p> 
<h2>Challenges with legacy architecture</h2> 
<p>As the Medidata portfolio of data offerings expanded and our strategy evolved to offering an end-to-end clinical data platform experience, the team recognized the need to evolve the data platform architecture. The following diagram shows Medidata’s legacy extract, transform, and load (ETL) architecture.</p> 
<p><img src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/11/26/BDB-5644-1.png" /></p> 
<p>The legacy system was built on scheduled batch jobs, which served Medidata well for a long time, but wasn’t designed for a unified view of data across a growing ecosystem. Batch jobs ran at different intervals, requiring scheduling buffers to make sure upstream jobs completed on time. As data volume grew, these schedules became more complex, introducing latency between ingestion and processing. Different consumers pulling from various data services also meant teams were continuously building new pipelines for each delivery stack.</p> 
<p>As the number of pipelines grew, so did the maintenance burden. More operations meant more potential points of failure and more complex recovery. The observability systems were handling a growing volume of operational data, making root cause analysis more involved. Scaling for increased data volume required coordinated changes across all our systems.<br /> Additionally, having data pipelines and copies spread across different technologies and storage systems meant access controls had to be managed in multiple places. Propagating access control changes correctly across all systems added complexity for both consumers and producers.</p> 
<h2>Solution overview</h2> 
<p>With the advent of Clinical Data Studio (Medidata’s unified data management and analytics solution for clinical trials) and Data Connect (Medidata’s data solution for acquiring, transforming, and exchanging electronic health record (EHR) data across healthcare organizations), Medidata introduced a new world of data discovery, analysis, and integration to the life sciences industry powered by open source technologies and hosted on AWS. The following diagram illustrates the solution architecture.</p> 
<p><img src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/11/26/BDB-5644-2.png" /></p> 
<p>Fragmented batch ETL jobs were replaced by real-time <a href="https://flink.apache.org/" rel="noopener noreferrer" target="_blank">Apache Flink</a> streaming pipelines, an open source, distributed engine for stateful processing, and powered by <a href="https://aws.amazon.com/eks/" rel="noopener noreferrer" target="_blank">Amazon Elastic Kubernetes Service</a> (Amazon EKS), a fully managed Kubernetes service. The Flink jobs write to Apache Kafka running in <a href="https://aws.amazon.com/msk/" rel="noopener noreferrer" target="_blank">Amazon Managed Apache Kafka</a> (Amazon MSK), a streaming data service that manages Kafka infrastructure and operations, before landing in Iceberg tables backed by the <a href="https://aws.amazon.com/glue/" rel="noopener noreferrer" target="_blank">AWS Glue Data Catalog</a>, a centralized metadata repository for data assets. From this collection of Iceberg tables, a central, single source of data is now accessible from a variety of consumers without additional downstream processing, alleviating the need for custom pipelines to satisfy the requirements of downstream consumers. Through these fundamental architectural changes, the team at Medidata solved the issues presented by the legacy solution.</p> 
<h2>Data availability and consistency</h2> 
<p>With the introduction of the Flink jobs and Iceberg tables, the team was able to deliver a consistent view of their data across the Medidata data experience. Pipeline latency was reduced from numerous hours and sometimes days to minutes, helping Medidata customers realize a 99% performance gain from the data ingestion to the data analytics layers. Due to Iceberg’s interoperability, Medidata users saw the same view of the data regardless of where they viewed that data, minimizing the need for consumer-driven custom pipelines because Iceberg could plug into existing consumers.</p> 
<h2>Maintenance and durability</h2> 
<p>Iceberg’s interoperability provided a single copy of the data to satisfy their use cases, so the Medidata team could focus its observation and maintenance efforts on a five-times smaller subset of operations than previously required. Observability was enhanced by tapping into the various metadata components and metrics exposed by Iceberg and the Data Catalog. Quality management transformed from cross-system traces and queries to a single analysis of unified pipelines, with an added benefit of point in time data queries thanks to the <a href="https://iceberg.apache.org/spec/#overview" rel="noopener noreferrer" target="_blank">Iceberg snapshot feature</a>. Data volume increases are handled with out-of-box scaling supported by the entire infrastructure stack and AWS Glue Iceberg optimization features that include <a href="https://docs.aws.amazon.com/glue/latest/dg/compaction-management.html" rel="noopener noreferrer" target="_blank">compaction</a>, <a href="https://docs.aws.amazon.com/glue/latest/dg/snapshot-retention-management.html" rel="noopener noreferrer" target="_blank">snapshot retention</a>, and <a href="https://docs.aws.amazon.com/glue/latest/dg/orphan-file-deletion.html" rel="noopener noreferrer" target="_blank">orphan file deletion</a>, which provide a set-and-forget experience for solving a number of common Iceberg frustrations, such as the small file problem, orphan file retention, and query performance.</p> 
<h2>Security</h2> 
<p>With Iceberg at the center of its solution architecture, the Medidata team no longer had to spend the time building custom access control layers with enhanced security features at each data integration point. Iceberg on AWS centralizes the authorization layer using familiar systems such as <a href="https://aws.amazon.com/iam/" rel="noopener noreferrer" target="_blank">AWS Identity and Access Management</a> (IAM), providing a single and durable control for data access. The data also stays entirely within the Medidata virtual private cloud (VPC), further reducing the opportunity for unintended disclosures.</p> 
<h2>Conclusion</h2> 
<p>In this post, we demonstrated how legacy universe of consumer-driven custom ETL pipelines can be replaced with a scalable, high-performant streaming lakehouses. By putting Iceberg on AWS at the center of data operations, you can have a single source of data for your consumers.</p> 
<p>To learn more about Iceberg on AWS, refer to <a href="https://docs.aws.amazon.com/glue/latest/dg/table-optimizers.html" rel="noopener noreferrer" target="_blank">Optimizing Iceberg tables</a> and <a href="https://docs.aws.amazon.com/prescriptive-guidance/latest/apache-iceberg-on-aws/introduction.html" rel="noopener noreferrer" target="_blank">Using Apache Iceberg on AWS</a>.</p> 
<hr /> 
<h3>About the authors</h3> 
<div class="blog-author-box"> 
 <footer> 
  <div class="blog-author-box"> 
   <div class="blog-author-image">
    <img alt="" class="aligncenter size-full wp-image-85750" height="100" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/11/27/mike.png" width="100" />
   </div> 
   <h3 class="lb-h4">Mike Araujo</h3> 
   <p><a href="https://www.linkedin.com/in/mike-araujo-200628112/" rel="noopener" target="_blank">Mike</a> is a Principal Engineer at Medidata Solutions, working on building a next generation data and AI platform for clinical data and trials. By using the power of open source technologies such as Apache Kafka, Apache Flink, and Apache Iceberg, Mike and his team have enabled the delivery of billions of clinical events and data transformations in near real time to downstream consumers, applications, and AI agents. His core skills focus on architecting and building big data and ETL solutions at scale as well as their integration in agentic workflows.</p> 
  </div> 
  <div class="blog-author-box"> 
   <div class="blog-author-image">
    <img alt="" class="aligncenter size-full wp-image-35755" height="133" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2022/10/13/Sandeep.jpg" width="100" />
   </div> 
   <h3 class="lb-h4">Sandeep Adwankar</h3> 
   <p><a href="https://www.linkedin.com/in/adwankar/" rel="noopener" target="_blank">Sandeep</a> is a Senior Product Manager at AWS, who has driven feature launches across Amazon SageMaker, AWS Glue, and AWS Lake Formation. He has led initiatives in Amazon S3 Tables analytics, Iceberg compaction strategies, and AWS Glue Iceberg optimizations. His recent work focuses on generative AI and autonomous systems, including the AWS Glue Data Catalog model context protocol and Amazon Bedrock structured knowledge bases. Based in the California Bay Area, he works with customers around the globe to translate business and technical requirements into products that accelerate their business outcomes.</p> 
  </div> 
  <div class="blog-author-box"> 
   <div class="blog-author-image">
    <img alt="" class="aligncenter size-full wp-image-85753" height="133" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/11/27/ibeatty.jpg" width="100" />
   </div> 
   <h3 class="lb-h4">Ian Beatty</h3> 
   <p><a href="https://www.linkedin.com/in/ian-beatty-02373459/" rel="noopener" target="_blank">Ian</a> is a Technical Account Manager at AWS, where he specializes in supporting independent software vendor (ISV) customers in the healthcare and life sciences (HCLS) and financial services industry (FSI) sectors. Based in the Rochester, NY area, Ian helps ISV customers navigate their cloud journey by maintaining resilient and optimized workloads on AWS. With over a decade of experience building on AWS since 2014, he brings deep technical expertise from his previous roles as an AWS Architect and DevSecOps team lead for SaaS ISVs before joining AWS more than 3 years ago.</p> 
  </div> 
  <div class="blog-author-box"> 
   <div class="blog-author-image">
    <img alt="" class="aligncenter size-full wp-image-85752" height="133" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/11/27/ashchenn-1.jpg" width="100" />
   </div> 
   <h3 class="lb-h4">Ashley Chen</h3> 
   <p><a href="https://www.linkedin.com/in/ashley-hy-chen/" rel="noopener" target="_blank">Ashley</a> is a Solutions Architect at AWS based in Washington D.C. She supports independent software vendor (ISV) customers in the healthcare and life sciences industries, focusing on customer enablement, generative AI applications, and container workloads.</p> 
  </div> 
 </footer> 
</div>
