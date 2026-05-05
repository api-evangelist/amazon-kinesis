---
title: "Perform Amazon Kinesis load testing with Locust"
url: "https://aws.amazon.com/blogs/big-data/perform-amazon-kinesis-load-testing-with-locust/"
date: "Thu, 10 Aug 2023 14:50:22 +0000"
author: "Luis Morales"
feed_url: "https://aws.amazon.com/blogs/big-data/category/analytics/amazon-kinesis/kinesis-data-firehose/feed/"
---
<p><em><strong>February 9, 2024:</strong> Amazon Kinesis Data Firehose has been renamed to Amazon Data Firehose. Read the <a href="https://aws.amazon.com/about-aws/whats-new/2024/02/amazon-data-firehose-formerly-kinesis-data-firehose/" rel="noopener" target="_blank">AWS What’s New post</a> to learn more.</em></p> 
<p>Building a streaming data solution requires thorough testing at the scale it will operate in a production environment. Streaming applications operating at scale often handle large volumes of up to GBs per second, and it’s challenging for developers to simulate high-traffic <a href="https://aws.amazon.com/kinesis/" rel="noopener" target="_blank">Amazon Kinesis</a>-based applications to generate such load easily.</p> 
<p><a href="https://aws.amazon.com/kinesis/streams/" rel="noopener" target="_blank">Amazon Kinesis Data Streams</a> and <a href="https://aws.amazon.com/kinesis/firehose" rel="noopener" target="_blank">Amazon Kinesis Data Firehose</a> are capable of capturing and storing terabytes of data per hour from numerous sources. Creating Kinesis data streams or Firehose delivery streams is straightforward through the <a href="http://aws.amazon.com/console" rel="noopener" target="_blank">AWS Management Console</a>, <a href="http://aws.amazon.com/cli" rel="noopener" target="_blank">AWS Command Line Interface</a> (AWS CLI), or Kinesis API. However, generating a continuous stream of test data requires a custom process or script to run continuously. Although the <a href="https://github.com/awslabs/amazon-kinesis-data-generator" rel="noopener" target="_blank">Amazon Kinesis Data Generator</a> (KDG) provides a user-friendly UI for this purpose, it has some limitations, such as bandwidth constraints and increased round trip latency. (For more information on the KDG, refer to <a href="https://aws.amazon.com/blogs/big-data/test-your-streaming-data-solution-with-the-new-amazon-kinesis-data-generator/" rel="noopener" target="_blank">Test Your Streaming Data Solution with the New Amazon Kinesis Data Generator</a>.)</p> 
<p>To overcome these limitations, this post describes how to use <a href="https://locust.io/" rel="noopener" target="_blank">Locust</a>, a modern load testing framework, to conduct large-scale load testing for a more comprehensive evaluation of the streaming data solution.</p> 
<h2>Overview</h2> 
<p>This project emits temperature sensor readings via Locust to Kinesis. We set up the <a href="http://aws.amazon.com/ec2" rel="noopener" target="_blank">Amazon Elastic Compute Cloud</a> (Amazon EC2) Locust instance via the <a href="https://aws.amazon.com/cdk/" rel="noopener" target="_blank">AWS Cloud Development Kit</a> (AWS CDK) to load test Kinesis-based applications. You can access the Locust dashboard to perform and observe the load test and connect via <a href="https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html" rel="noopener" target="_blank">Session Manager</a>, a capability of <a href="https://aws.amazon.com/systems-manager/" rel="noopener" target="_blank">AWS Systems Manager</a>, for configuration changes. The following diagram illustrates this architecture.</p> 
<p><img alt="Architecture overview" class="alignnone wp-image-51798 size-full" height="1162" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/08/07/BDB-3247-image001.jpg" width="1562" /></p> 
<p>In our testing with the largest recommended instance (c7g.16xlarge), the setup was capable of emitting over 1 million events per second to Kinesis data streams in on-demand capacity mode, with a batch size (simulated users per Locust user) of 500. You can find more details on what this means and how to configure the load test later in this post.</p> 
<h2>Locust overview</h2> 
<p>Locust is an open-source, scriptable, and scalable performance testing tool that allows you to define user behavior using Python code. It offers an easy-to-use interface, making it developer-friendly and highly expandable. With its distributed and scalable design, Locust can simulate millions of simultaneous users to mimic real user behavior during a performance test.</p> 
<p>Each Locust user represents a scenario or a specific set of actions that a real user might perform on your system. When you run a performance test with Locust, you can specify the number of concurrent Locust users you want to simulate, and Locust will create an instance for each user, allowing you to assess the performance and behavior of your system under different user loads.</p> 
<p>For more information on Locust, refer to the&nbsp;<a href="https://docs.locust.io/en/latest/what-is-locust.html" rel="noopener" target="_blank">Locust documentation</a>.</p> 
<h2>Prerequisites</h2> 
<p>To get started, clone or download the code from the&nbsp;<a href="https://github.com/aws-samples/amazon-kinesis-load-testing-with-locust" rel="noopener" target="_blank">GitHub repository</a>.</p> 
<h2>Test locally</h2> 
<p>To test Locust out locally first before deploying it to the cloud, you have to install the necessary Python dependencies.&nbsp;If you’re new to Python, refer the <a href="https://github.com/aws-samples/amazon-kinesis-load-testing-with-locust/blob/main/README.md" rel="noopener" target="_blank">README</a> for more information on getting started.</p> 
<p>Navigate to the <code>load-test</code> directory and run the following code:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">pip install -r requirements.txt</code></pre> 
</div> 
<p>To send events to a Kinesis data stream from your local machine, you will need to have AWS credentials. For more information, refer to&nbsp;<a href="https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html" rel="noopener" target="_blank">Configuration and credential file settings</a>.</p> 
<p>To perform the test locally, stay in the&nbsp;<code>load-test</code> directory and run the following code:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">locust -f&nbsp;locust-load-test.py</code></pre> 
</div> 
<p>You can now access the Locust dashboard via&nbsp;<a href="http://0.0.0.0:8089/" rel="noopener" target="_blank">http://0.0.0.0:8089/</a>.&nbsp;Enter the number of Locust users, the spawn rate (users added per second), and the target Amazon Kinesis data stream name for <strong>Host</strong>. By default, it deploys the Kinesis data stream <code>DemoStream</code> that you can use for testing.</p> 
<p><img alt="Locust Dashboard - Enter details" class="alignnone wp-image-51799 size-full" height="1441" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/08/07/BDB-3247-image002.jpg" style="margin: 10px 0px 10px 0px;" width="2560" /></p> 
<p>To see the generated events logged, run the following command, which filters only Locust and root logs (for example, no Botocore logs):</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">locust -f&nbsp;locust-load-test.py --loglevel&nbsp;DEBUG 2&amp;gt;&amp;amp;1 | grep&nbsp;-E&nbsp;"(locust|root)"</code></pre> 
</div> 
<h2>Set up resources with the AWS CDK</h2> 
<p>The GitHub repository contains the AWS CDK code to create all the necessary resources for the load test. This removes opportunities for manual error, increases efficiency, and ensures consistent configurations over time. To deploy the resources, complete the following steps:</p> 
<ol> 
 <li>If not already downloaded, clone the GitHub repository to your local computer using the following command:</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-bash">git clone https://github.com/aws-samples/amazon-kinesis-load-testing-with-locust</code></pre> 
</div> 
<ol start="2"> 
 <li><a href="https://nodejs.org/en/download/" rel="noopener" target="_blank">Download</a> and install the latest Node.js.</li> 
 <li>Navigate to the root folder of the project and run the following command to install the latest version of AWS CDK:</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-bash">npm install -g aws-cdk</code></pre> 
</div> 
<ol start="4"> 
 <li>Install the necessary dependencies:</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-bash">npm install</code></pre> 
</div> 
<ol start="5"> 
 <li>Run&nbsp;cdk bootstrap to initialize the AWS CDK environment in your AWS account. Replace your AWS account ID and Region before running the following command:</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-bash">cdk bootstrap</code></pre> 
</div> 
<p>To learn more about the bootstrapping process, refer to <a href="https://docs.aws.amazon.com/cdk/v2/guide/bootstrapping.html" rel="noopener" target="_blank">Bootstrapping</a>.</p> 
<ol start="6"> 
 <li>After the dependencies are installed, you can run the following command to deploy the stack of the AWS CDK template, which sets up the infrastructure within 5 minutes:</li> 
</ol> 
<div class="hide-language"> 
 <pre><code class="lang-bash">cdk deploy</code></pre> 
</div> 
<p>The template sets up the Locust EC2 test instance, which is by default a c7g.xlarge instance, which at the time of publishing costs approximately $0.145 per hour in us-east-1. To find the most accurate pricing information, see Amazon EC2 On-Demand Pricing. You can find more details on how to change your instance size according to your scale of load testing later in this post.</p> 
<p>It’s crucial to consider that the expenses incurred during load testing are not solely attributed to EC2 instance costs, but also heavily influenced by data transfer costs.</p> 
<h3>Accessing the Locust dashboard</h3> 
<p>You can access the dashboard by using the AWS CDK output&nbsp;<code>KinesisLocustLoadTestingStack.locustdashboardurl</code> to open the dashboard, for example <code>http://1.2.3.4:8089</code>.</p> 
<p>The Locust dashboard is password protected. By default, it’s set to user name <code>locust-user</code> and password <code>locust-dashboard-pwd</code>.</p> 
<p>With the default configuration, you can achieve up to 15,000 emitted events per second. Enter the number of Locust users (times the batch size), the spawn rate (users added per second), and the target Kinesis data stream name for <strong>Host</strong>.</p> 
<p><img alt="Locust Dashboard - Enter details" class="alignnone wp-image-51800 size-full" height="1004" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/08/07/BDB-3247-image003.jpg" style="margin: 10px 0px 10px 0px;" width="1795" /></p> 
<p>After you have started the load test, you can look at the load test on the <strong>Charts</strong> tab.</p> 
<p><img alt="Locust Dashboard - Charts" class="alignnone wp-image-51801 size-full" height="891" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/08/07/BDB-3247-image004.jpg" style="margin: 10px 0px 10px 0px;" width="1770" /></p> 
<p>You can also monitor the load test on the Kinesis Data Streams console by navigating to the stream that you are load testing. If you used the default settings, navigate to <code>DemoStream</code>. On the detail page, choose the <strong>Monitoring</strong> tab to see the ingested load.</p> 
<p><img alt="Kinesis Data Streams - Monitoring" class="alignnone wp-image-51802 size-full" height="546" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/08/07/BDB-3247-image005.jpg" style="margin: 10px 0px 10px 0px;" width="1695" /></p> 
<h2>Adapt workloads</h2> 
<p>By default, this project generates random temperature sensor readings for every sensor with the following format:</p> 
<div class="hide-language"> 
 <pre><code class="lang-json">{
    "sensorId": "bfbae19c-2f0f-41c2-952b-5d5bc6e001f1_1",
    "temperature": 147.24,
    "status": "OK",
    "timestamp": 1675686126310
}</code></pre> 
</div> 
<p>The project comes packaged with Faker, which you can use to adapt the payload to your needs. You just have to update the <code>generate_sensor_reading</code> function in the <code>locust-load-test.py</code> file:</p> 
<div class="hide-language"> 
 <pre class="unlimited-height-code"><code class="lang-python">class SensorAPIUser(KinesisBotoUser):
    # ...

    def generate_sensor_reading(self, sensor_id, sensor_reading):
        current_temperature = round(10 + random.random() * 170, 2)

        if current_temperature &gt; 160:
            status = "ERROR"
        elif current_temperature &gt; 140 or random.randrange(1, 100) &gt; 80:
            status = random.choice(["WARNING", "ERROR"])
        else:
            status = "OK"

        return {
            'sensorId': f"{sensor_id}_{sensor_reading}",
            'temperature': current_temperature,
            'status': status,
            'timestamp': round(time.time()*1000)
        }

    # ...</code></pre> 
</div> 
<h2>Change configurations</h2> 
<p>After the initial deployment of the load testing tool, you can change configuration in two ways:</p> 
<ol> 
 <li>Connect to the EC2 instance, make any configuration and code changes, and restart the Locust process</li> 
 <li>Change the configuration and load testing code locally and redeploy it via <code>cdk deploy</code></li> 
</ol> 
<p>The first option helps you iterate more quickly on the remote instance without a need to redeploy. The latter uses the infrastructure as code (IaC) approach and makes sure that your configuration changes can be committed to your source control system. For a fast development cycle, it’s recommended to test your load test configuration locally first, connect to your instance to apply the changes, and after successful implementation, codify it as part of your IaC repository and then redeploy.</p> 
<p>Locust is created on the EC2 instance as a <a href="https://systemd.io/" rel="noopener" target="_blank">systemd</a> service and can therefore be controlled with <a href="https://www.commandlinux.com/man-page/man1/systemctl.1.html" rel="noopener" target="_blank">systemctl</a>. If you want to change the configuration of Locust as needed without redeploying the stack, you can connect to the instance via Systems Manager, navigate to the project directory on <code>/usr/local/load-test</code>, change the <code>locust.env</code> file, and restart the service by running <code>sudo systemctl restart locust</code>.</p> 
<h2>Large-scale load testing</h2> 
<p>This setup is capable of emitting over 1 million events per second to Kinesis data stream, with a batch size of 500 and 64 secondaries on a&nbsp;c7g.16xlarge.</p> 
<p>To achieve peak performance with Locust and Kinesis, keep the following in mind:</p> 
<ul> 
 <li><strong>Instance size</strong> – Your performance is bound by the underlying EC2 instance, so refer to <a href="https://github.com/aws-samples/amazon-kinesis-load-testing-with-locust#ec2-instance-type" rel="noopener" target="_blank">EC2 instance type</a> for more information about scaling. To set the correct instance size, you can configure the instance size in the file <a href="https://github.com/aws-samples/amazon-kinesis-load-testing-with-locust/blob/main/infrastructure/kinesis-load-testing-with-locust.ts#L27" rel="noopener" target="_blank">kinesis-locust-load-testing.ts</a>.</li> 
 <li><strong>Number of secondaries</strong> – Locust benefits from a distributed setup. Therefore, the setup spins up a primary, which does the coordination, and multiple secondaries, which do the actual work. To fully take advantage of the cores, you should specify one secondary per core. You can configure the number in the <a href="https://github.com/aws-samples/amazon-kinesis-load-testing-with-locust/blob/main/load-test/locust.env#L1" rel="noopener" target="_blank">locust.env</a> file.</li> 
 <li><strong>Batch size</strong> – The amount of Kinesis data stream events you can send per Locust user is limited due to the resource overhead of switching Locust users and threads. To overcome this, you can configure a batch size to define how much users are simulated per Locust user. These are sent as a Kinesis data stream <a href="https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/kinesis/client/put_records.html" rel="noopener" target="_blank">put_records</a> call. You can configure the number in the <a href="https://github.com/aws-samples/amazon-kinesis-load-testing-with-locust/blob/main/load-test/locust.env#L2" rel="noopener" target="_blank">locust.env</a> file.</li> 
</ul> 
<p>This setup is capable of emitting over 1 million events per second to the Kinesis data stream, with a batch size of 500 and 64 secondaries on a c7g.16xlarge instance.</p> 
<p><img alt="Locust Dashboard - Large Scale Load Test Charts" class="alignnone wp-image-51803 size-full" height="1299" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/08/07/BDB-3247-image006.jpg" style="margin: 10px 0px 10px 0px;" width="2560" /></p> 
<p>You can observe this on the <strong>Monitoring</strong> tab for the Kinesis data stream as well.</p> 
<p><img alt="Kinesis Data Stream - Large Scale Load Test Monitoring" class="alignnone wp-image-51804 size-full" height="549" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/08/07/BDB-3247-image007.jpg" style="margin: 10px 0px 10px 0px;" width="1698" /></p> 
<h2>Clean up</h2> 
<p>In order to not incur any unnecessary costs, delete the stack by running the following code:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">cdk destroy</code></pre> 
</div> 
<h2>Summary</h2> 
<p>Kinesis is already popular for its ease of use among users building streaming applications. With this load testing capability using Locust, you can now test your workloads in a more straightforward and faster way. Visit the <a href="https://github.com/aws-samples/amazon-kinesis-load-testing-with-locust" rel="noopener" target="_blank">GitHub repo</a> to embark on your testing journey.</p> 
<p>The project is licensed under the Apache 2.0 license, providing the freedom to clone and modify it according to your needs. Furthermore, you can contribute to the project by submitting issues or pull requests via GitHub, fostering collaboration and improvement in the testing ecosystem.</p> 
<hr /> 
<h3>About the author</h3> 
<p><img alt="" class="wp-image-51874 size-full alignleft" height="133" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/08/08/Luis-100.jpg" width="100" /></p> 
<p><strong>Luis Morales</strong> works as Senior Solutions Architect with digital native businesses to support them in constantly reinventing themselves in the cloud. He is passionate about software engineering, cloud-native distributed systems, test-driven development, and all things code and security</p>
