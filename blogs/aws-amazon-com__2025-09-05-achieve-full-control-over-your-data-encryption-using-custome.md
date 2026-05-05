---
title: "Achieve full control over your data encryption using customer managed keys in Amazon Managed Service for Apache Flink"
url: "https://aws.amazon.com/blogs/big-data/achieve-full-control-over-your-data-encryption-using-customer-managed-keys-in-amazon-managed-service-for-apache-flink/"
date: "Fri, 05 Sep 2025 20:33:20 +0000"
author: "Lorenzo Nicora"
feed_url: "https://aws.amazon.com/blogs/big-data/category/analytics/amazon-kinesis/feed/"
---
<p>Encryption of both data at rest and in transit is a non-negotiable feature for most organizations. Furthermore, organizations operating in highly regulated and security-sensitive environments—such as those in the financial sector—often require full control over the cryptographic keys used for their workloads.</p> 
<p><a href="https://aws.amazon.com/managed-service-apache-flink/" rel="noopener noreferrer" target="_blank">Amazon Managed Service for Apache Flink</a> makes it straightforward to process real-time data streams with robust security features, including encryption by default to help protect your data in transit and at rest. The service removes the complexity of managing the key lifecycle and controlling access to the cryptographic material.</p> 
<p>If you need to retain full control over your key lifecycle and access, Managed Service for Apache Flink now supports the use of customer managed keys (CMKs) stored in <a href="https://aws.amazon.com/kms/" rel="noopener noreferrer" target="_blank">AWS Key Management Service</a> (AWS KMS) for encrypting application data.</p> 
<p>This feature helps you manage your own encryption keys and key policies, so you can meet strict compliance requirements and maintain complete control over sensitive data. With CMK integration, you can take advantage of the scalability and ease of use that Managed Service for Apache Flink offers, while meeting your organization’s security and compliance policies.</p> 
<p>In this post, we explore how the CMK functionality works with Managed Service for Apache Flink applications, the use cases it unlocks, and key considerations for implementation.</p> 
<h2>Data encryption in Managed Service for Apache Flink</h2> 
<p>In Managed Service for Apache Flink, there are multiple aspects where data should be encrypted:</p> 
<ul> 
 <li><strong>Data at rest directly managed by the service</strong> – Durable application storage (checkpoints and snapshots) and running application state storage (disk volumes used by <a href="https://github.com/facebook/rocksdb/wiki" rel="noopener" target="_blank">RocksDB</a> state backend) are automatically encrypted</li> 
 <li><strong>Data in transit internal to the Flink cluster</strong> – Automatically encrypted using TLS/HTTPS</li> 
 <li><strong>Data in transit to and at rest in external systems that your Flink application accesses</strong> – For example, an <a href="https://aws.amazon.com/msk/" rel="noopener noreferrer" target="_blank">Amazon Managed Streaming for Apache Kafka</a> (Amazon MSK) topic through the Kafka connector or calling an endpoint through a custom <a href="https://nightlies.apache.org/flink/flink-docs-release-1.20/docs/dev/datastream/operators/asyncio/" rel="noopener noreferrer" target="_blank">AsyncIO</a>); encryption depends on the external service, user settings, and code</li> 
</ul> 
<p>For data at rest managed by the service, checkpoints, snapshots, and running application state storage are encrypted by default using AWS owned keys. If your security requirements require you to directly control the encryption keys, you can use the CMK held in AWS KMS.</p> 
<h2>Key components and roles</h2> 
<p>To understand how CMKs work in Managed Service for Apache Flink, we first need to introduce the components and roles involved in managing and running an application using CMK encryption:</p> 
<ul> 
 <li><strong>Customer managed key (CMK):</strong> 
  <ul> 
   <li>Resides in AWS KMS within the same AWS account as your application</li> 
   <li>Has an attached key policy that defines access permissions and usage rights to other components and roles</li> 
   <li>Encrypts both durable application storage (checkpoints and snapshots) and running application state storage</li> 
  </ul> </li> 
 <li><strong>Managed Service for Apache Flink application:</strong> 
  <ul> 
   <li>The application whose storage you want to encrypt using the CMK</li> 
   <li>Has an attached <a href="https://aws.amazon.com/iam/" rel="noopener noreferrer" target="_blank">AWS Identity and Access Management</a> (IAM) execution role that grants permissions to access external services</li> 
   <li>The execution role doesn’t have to provide any specific permissions to use the CMK for encryption operations</li> 
  </ul> </li> 
 <li><strong>Key administrator:</strong> 
  <ul> 
   <li>Manages the CMK lifecycle (creation, rotation, policy updates, and so on)</li> 
   <li>Can be an IAM user or IAM role, and used by a human operator or by automation</li> 
   <li>Requires administrative access to the CMK</li> 
   <li>Permissions are defined by the attached IAM policies and the key policy</li> 
  </ul> </li> 
 <li><strong>Application operator:</strong> 
  <ul> 
   <li>Manages the application lifecycle (start/stop, configuration updates, snapshot management, and so on)</li> 
   <li>Can be an IAM User or IAM role, and used by a human operator or by automation</li> 
   <li>Requires permissions to manage the Flink application and use the CMK for encryption operations</li> 
   <li>Permissions are defined by the attached IAM policies and the key policy</li> 
  </ul> </li> 
</ul> 
<p>The following diagram illustrates the solution architecture.</p> 
<p><img alt="Actors" class="aligncenter size-full wp-image-82815" height="1184" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/08/29/BDB-5441-01-actors.png" width="1490" /></p> 
<h2>Enabling CMK following the principle of least privilege</h2> 
<p>When deploying applications in production environments or handling sensitive data, you should follow the principle of least privilege. CMK support in Managed Service for Apache Flink has been designed with this principle in mind, so each component receives only the minimum permissions necessary to function.</p> 
<p>For detailed information about the permissions required by the application operator and key policy configurations, refer to <a href="https://docs.aws.amazon.com/managed-flink/latest/java/key-management-flink.html" rel="noopener noreferrer" target="_blank">Key management in Amazon Managed Service for Apache Flink</a>. Although these policies might appear complex at first glance, this complexity is intentional and necessary. For more details about the requirements for implementing the most restrictive key management possible while maintaining functionality, refer to <a href="https://docs.aws.amazon.com/kms/latest/developerguide/least-privilege.html" rel="noopener noreferrer" target="_blank">Least-privilege permissions</a>.</p> 
<p>For this post, we highlight some important points about CMK permissions:</p> 
<ul> 
 <li><strong>Application execution role</strong> – Requires no additional permissions to use a CMK. You don’t need to change the permissions of an existing application; the service handles CMK operations transparently during runtime.</li> 
 <li><strong>Application operator permissions </strong>– The operator is the user or role who controls the application lifecycle. For the permissions required to operate an application that uses CMK encryption, refer to <a href="https://docs.aws.amazon.com/managed-flink/latest/java/key-management-flink.html" rel="noopener noreferrer" target="_blank">Key management in Amazon Managed Service for Apache Flink</a>. In addition to these permissions, an operator normally has permissions on actions with the <code>kinesisanalytics</code> prefix. It is a best practice to restrict these permissions to a specific application defining the <code>Resource</code>. The operator must also have the <code>iam:PassRole</code> permission to pass the service execution role to the application.</li> 
</ul> 
<p>To simplify managing the permissions of the operator, we recommend creating two separate IAM policies, to be attached to the operator’s role or user:</p> 
<ul> 
 <li>A base operator policy defining the basic permissions to operate the application lifecycle without a CMK</li> 
 <li>An additional CMK operator policy that adds permissions to operate the application with a CMK</li> 
</ul> 
<p>The following IAM policy example illustrates the permissions that should be included in the base operator policy:</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Allow Managed Flink operations",
      "Effect": "Allow",
      "Action": "kinesisanalytics:*",
      "Resource": "arn:aws:kinesisanalytics:<span style="color: #ff0000;">&lt;region&gt;</span>:<span style="color: #ff0000;">&lt;account-id&gt;</span>:application/<span style="color: #ff0000;">MyApplication</span>"
    },
    {
      "Sid": "Allow passing service execution role",
      "Effect": "Allow",
      "Action": [
        "iam:PassRole"
      ],
      "Resource": "arn:aws:iam::<span style="color: #ff0000;">&lt;account-id&gt;</span>:role/<span style="color: #ff0000;">MyApplicationRole</span>"
    },
  ]
} </code></pre> 
</div> 
<p>Refer to <a href="https://docs.aws.amazon.com/managed-flink/latest/java/manage-cmk-api.html#create-cmk-kms-api-caller-permissions" rel="noopener noreferrer" target="_blank">Application lifecycle operator (API caller) permissions</a> for the permissions to be included with the additional CMK operator policy.</p> 
<p>Separating these two policies has an additional benefit of simplifying the process of setting up an application for the CMK, due to the dependencies we illustrate in the following section.</p> 
<h3>Dependencies between the key policy and CMK operator policy</h3> 
<p>If you carefully observe the operator’s permissions and the key policy explained in <a href="https://docs.aws.amazon.com/managed-flink/latest/java/manage-cmk-api.html#create-cmk-kms-key-policy" rel="noopener noreferrer" target="_blank">Create a KMS key policy</a>, you will notice some interdependencies, illustrated by the following diagram.</p> 
<p><img alt="Dependencies" class="aligncenter size-full wp-image-82816" height="1194" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/08/29/BDB-5441-02-dependencies.png" width="1108" /></p> 
<p>In particular, we highlight the following:</p> 
<ul> 
 <li><strong>CMK key policy dependencies</strong> – The CMK policy requires references to both the application Amazon Resource Name (ARN) and the key administrator or operator IAM roles or users. This policy must be defined at key creation time by the key administrator.</li> 
 <li><strong>IAM policy dependencies </strong>– The operator’s IAM policy must reference both the application ARN and the CMK key itself. The operator role is responsible for various tasks, including configuring the application to use the CMK.</li> 
</ul> 
<p>To properly follow the principle of least privilege, each component requires the others to exist before it can be correctly configured. This necessitates a carefully orchestrated deployment sequence.</p> 
<p>In the following section, we demonstrate the precise order required to resolve these dependencies while maintaining security best practices.</p> 
<h3>Sequence of operations to create a new application with a CMK</h3> 
<p>When deploying a new application that uses CMK encryption, we recommend following this sequenced approach to resolve dependency conflicts while maintaining security best practices:</p> 
<ol> 
 <li>Create the operator IAM role or user with a base policy that includes application lifecycle permissions. Do not include CMK permissions at this stage, because the key doesn’t exist yet.</li> 
 <li>The operator creates the application using the default AWS owned key. Keep the application in a stopped state to prevent data creation—there should be no data at rest to encrypt during this phase.</li> 
 <li>Create the key administrator IAM role or user, if not already available, with permissions to create and manage KMS keys. Refer to <a href="https://docs.aws.amazon.com/kms/latest/developerguide/iam-policies.html" rel="noopener noreferrer" target="_blank">Using IAM policies with AWS KMS</a> for detailed permission requirements.</li> 
 <li>The key administrator creates the CMK in AWS KMS. At this point, you have the required components for the key policy: application ARN, operator IAM role or user ARN, and key administrator IAM role or user ARN.</li> 
 <li>Create and attach to the operator an additional IAM policy that includes the CMK-specific permissions. See <a href="https://docs.aws.amazon.com/managed-flink/latest/java/manage-cmk-api.html#create-cmk-kms-api-caller-permissions" rel="noopener noreferrer" target="_blank">Application lifecycle operator (API caller) permissions</a> for the complete operator policy definition.</li> 
 <li>The operator can now modify the application configuration using the <a href="https://docs.aws.amazon.com/managed-flink/latest/apiv2/API_UpdateApplication.html" rel="noopener noreferrer" target="_blank"><code>UpdateApplication</code> action</a>, to enable CMK encryption, as illustrated in the following section.</li> 
 <li>The application is now ready to run with all data at rest encrypted using your CMK.</li> 
</ol> 
<h3>Enable the CMK with UpdateApplication</h3> 
<p>You can configure a Managed Service for Apache Flink application to use a CMK using the <a href="http://aws.amazon.com/console" rel="noopener noreferrer" target="_blank">AWS Management Console</a>, the <a href="https://docs.aws.amazon.com/managed-flink/latest/apiv2/Welcome.html" rel="noopener" target="_blank">AWS API</a>, <a href="http://aws.amazon.com/cli" rel="noopener noreferrer" target="_blank">AWS Command Line Interface</a> (AWS CLI), or infrastructure as code (IaC) tools like the <a href="https://aws.amazon.com/cdk/" rel="noopener noreferrer" target="_blank">AWS Cloud Development Kit</a> (AWS CDK) or <a href="https://aws.amazon.com/cloudformation/">AWS CloudFormation</a> templates.</p> 
<p>When setting up CMK encryption in a production environment, you will probably use an automation tool rather than the console. These tools eventually use the AWS API under the hood, and the <code>UpdateApplication</code> action of the <code>kinesisanalyticsv2</code> API in particular. In this post, we analyze the additions to the API that you can use to control the encryption configuration.</p> 
<p>An additional top-level block <code>ApplicationEncryptionConfigurationUpdate</code> has been added to the <code>UpdateApplication</code> request payload. With this block, you can enable and disable the CMK.</p> 
<p>You must add the following block to the <code>UpdateApplication</code> request:</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">{
  "ApplicationEncryptionConfigurationUpdate": {
    "KeyTypeUpdate": "CUSTOMER_MANAGED_KEY",
    "KeyIdUpdate": "arn:aws:kms:us-east-1:123456789012:key/01234567-99ab-cdef-0123-456789abcdef"
  }
}</code></pre> 
</div> 
<p>The <code>KeyIdUpdate</code> value can be the key ARN, key ID, key alias name, or key alias ARN.</p> 
<h3>Disable the CMK</h3> 
<p>Similarly, the following requests disable the CMK, switching back to the default AWS owned key:</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">{
  "ApplicationEncryptionConfigurationUpdate": {
    "KeyTypeUpdate": "AWS_OWNED_KEY"
  }
}</code></pre> 
</div> 
<h3>Enable the CMK with CreateApplication</h3> 
<p>Theoretically, you can enable the CMK directly when you first create the application using the <a href="https://docs.aws.amazon.com/managed-flink/latest/apiv2/API_CreateApplication.html" rel="noopener noreferrer" target="_blank"><code>CreateApplication</code></a> action.</p> 
<p>A top-level block <code>ApplicationEncryptionConfiguration</code> has been added to the <code>CreateApplication</code> request payload, with a syntax similar to <a href="https://docs.aws.amazon.com/managed-flink/latest/apiv2/API_UpdateApplication.html" rel="noopener noreferrer" target="_blank"><code>UpdateApplication</code></a>.</p> 
<p>However, due to the interdependencies described in the previous section, you will most often create an application with the default AWS owned key and later use <code>UpdateApplication</code> to enable the CMK.</p> 
<p>If you omit <code>ApplicationEncryptionConfiguration</code> when you create the application, the default behavior is using the AWS owned key, for backward compatibility.</p> 
<h2>Sample CloudFormation templates to create IAM roles and the KMS key</h2> 
<p>The process you use to create the roles and key and configure the application to use the CMK will vary, depending on the automation you use and your approval and security processes. Any automation example we can provide will likely not fit your processes or tooling.</p> 
<p>However, the following <a href="https://github.com/aws-samples/amazon-managed-service-for-apache-flink-examples/tree/main/infrastructure/CMK" rel="noopener noreferrer" target="_blank">GitHub repository</a> provides some example CloudFormation templates to generate some of the IAM policies and the KMS key with the correct key policy:</p> 
<ul> 
 <li><strong>IAM policy for the key administrator</strong> – Allows managing the key</li> 
 <li><strong>Base IAM policy for the operator</strong> – Allows managing the normal application lifecycle operations without the CMK</li> 
 <li><strong>CMK IAM policy for the operator</strong> – Provides additional permissions required to manage the application lifecycle when the CMK is enabled</li> 
 <li><strong>KMS key policy</strong> – Allows the application to encrypt and decrypt the application state and the operator to manage the application operations</li> 
</ul> 
<h2>CMK operations</h2> 
<p>We have described the process of creating a new Managed Service for Apache Flink application with CMK. Let’s now examine other common operations you can perform.</p> 
<p>Changes to the encryption key become effective when the application is restarted. If you update the configuration of a running application, this causes the application to restart and the new key to be used immediately. Conversely, if you change the key of a <code>READY</code> (not running) application, the new key is not actually used until the application is restarted.</p> 
<h3>Enable a CMK on an existing application</h3> 
<p>If you have an application running with an AWS owned key, the process is similar to what we described for creating new applications. In this case, you already have a running application state and older snapshots that are encrypted using the AWS owned key.</p> 
<p>Also, if you have a running application, you probably already have an operator role with an IAM policy that you can use to control the operator lifecycle.</p> 
<p>The sequence of steps to enable a CMK on an existing and running application is as follows:</p> 
<ol> 
 <li>If you don’t already have one, create a key administrator IAM role or user with permissions to create and manage keys in AWS KMS. See <a href="https://docs.aws.amazon.com/kms/latest/developerguide/iam-policies.html" rel="noopener noreferrer" target="_blank">Using IAM policies with AWS KMS</a> for more details about the permissions required to manage keys.</li> 
 <li>The key administrator creates the CMK. The key policy references the application ARN, the operator’s ARN, and the key administrator’s role or user ARN.</li> 
 <li>Create an additional IAM policy that allows the use of the CMK and attach this policy to the operator. Alternatively, modify the operator’s existing IAM policy by adding these permissions.</li> 
 <li>Finally, the operator can update the application and enable the CMK.The following diagram illustrates the process that occurs when you execute an <a href="https://docs.aws.amazon.com/managed-flink/latest/apiv2/API_UpdateApplication.html" rel="noopener noreferrer" target="_blank"><code>UpdateApplication</code></a> action on the running application to enable a CMK. <p><img alt="Enabling CMK on an existing application" class="aligncenter size-full wp-image-82817" height="676" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/08/29/BDB-5441-03-enable-cmk-on-existing-application.png" width="1164" /></p> <p>The workflow consists of the following steps:</p></li> 
 <li>When you update the application to set up the CMK, the following happens: 
  <ol type="a"> 
   <li>The application running state, at the moment it is encrypted with the AWS owned key, is saved in a snapshot while the application is stopped. This snapshot is encrypted with the default AWS owned key. The running application state storage is volatile and destroyed when the application is stopped.</li> 
   <li>The application is redeployed, restoring the snapshot into the running application state.</li> 
   <li>The running application state storage is now encrypted with the CMK.</li> 
  </ol> </li> 
 <li>New snapshots created from this point on are encrypted using the CMK.</li> 
 <li>You will probably want to delete all the old snapshots, including the one created automatically by the <code>UpdateApplication</code> that enabled the CMK, because they are all encrypted using the AWS owned key.</li> 
</ol> 
<h3>Rotate the encryption key</h3> 
<p>As with any cryptographic key, it’s a best practice to rotate the key periodically for enhanced security. Managed Service for Apache Flink does not support <a href="https://docs.aws.amazon.com/kms/latest/developerguide/rotate-keys.html" rel="noopener noreferrer" target="_blank">AWS KMS automatic key rotation,</a> so you have two primary options for rotating your CMK.</p> 
<h4>Option 1: Create a new CMK and update the application</h4> 
<p>The first approach involves creating an entirely new KMS key and then updating your application configuration to use the new key. This method provides a clean separation between the old and new encryption keys, making it easier to track which data was encrypted with which key version.</p> 
<p>Let’s assume you have a running application using CMK#1 (the current key) and want to rotate to CMK#2 (the new key) for enhanced security:</p> 
<ul> 
 <li><strong>Prerequisites and preparation </strong>– Before initiating the key rotation process, you must update the operator’s IAM policy to include permissions for both CMK#1 and CMK#2. This dual-key access supports uninterrupted operation during the transition period. After the application configuration has been successfully updated and verified, you can safely remove all permissions to CMK#1.</li> 
 <li><strong>Application update process </strong>– The <code>UpdateApplication</code> operation used to configure CMK#2 automatically triggers an application restart. This restart mechanism makes sure both the application’s running state and any newly created snapshots are encrypted using the new CMK#2, providing immediate security benefits from the updated encryption key.</li> 
 <li><strong>Important security considerations </strong>– Existing snapshots, including the automatic snapshot created during the CMK update process, remain encrypted with the original CMK#1. For complete security hygiene and to minimize your cryptographic footprint, consider deleting these older snapshots after verifying that your application is functioning correctly with the new encryption key.</li> 
</ul> 
<p>This approach provides a clean separation between old and new encrypted data while maintaining application availability throughout the key rotation process.</p> 
<h4>Option 2: Rotate the key material of the existing CMK</h4> 
<p>The second option is to rotate the cryptographic material within your existing KMS key. For a CMK used for Managed Service for Apache Flink, we recommend using <a href="https://docs.aws.amazon.com/kms/latest/developerguide/rotating-keys-on-demand.html" rel="noopener noreferrer" target="_blank">on-demand key material rotation</a>.</p> 
<p>The benefit of this approach is simplicity: no change is required to the application configuration nor to the operator’s IAM permissions.</p> 
<h4>Important security considerations</h4> 
<p>The new encryption key is used by the Managed Service for Apache Flink application only after the next application restart. To make the new key material effective, immediately after the rotation, you need to stop and start using snapshots to preserve the application state or execute an <code>UpdateApplication</code>, which also forces a stop-and-restart. After the restart, you should consider deleting the old snapshots, including the one taken automatically in the last stop-and-restart.</p> 
<h3>Switch back to the AWS owned key</h3> 
<p>At any time, you can decide to switch back to using an AWS owned key. The application state is still encrypted, but using the AWS owned key instead of your CMK.</p> 
<p>If you are using the <code>UpdateApplication</code> API or AWS CLI command to switch back to CMK, you must explicitly pass <code>ApplicationEncryptionConfigurationUpdate</code>, setting the key type to <code>AWS_OWNED_KEY</code> as shown in the following snippet:</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">{
  "ApplicationEncryptionConfigurationUpdate": {
    "KeyTypeUpdate": "AWS_OWNED_KEY"
  }
}</code></pre> 
</div> 
<p>When you execute <code>UpdateApplication</code> to switch off the CMK, the operator must still have permissions on the CMK. After the application is successfully running using the AWS owned key, you can safely remove any CMK-related permissions from the operator’s IAM policy.</p> 
<h2>Test the CMK in development environments</h2> 
<p>In a production environment—or an environment containing sensitive data—you should follow the principle of least privilege and apply the restrictive permissions described so far.</p> 
<p>However, if you want to experiment with CMKs in a development setting, such as using the console, strictly following the production process might become cumbersome. In these environments, the roles of key administrator and operator are often filled by the same person.</p> 
<p>For testing purposes in development environments, you might want to use a permissive key policy like the following, so you can freely experiment with CMK encryption:</p> 
<div class="hide-language"> 
 <pre><code class="lang-code">{
  "Version": "2012-10-17",
  "Id": "key-policy-permissive-for-dev-only",
  "Statement": [
    {
      "Sid": "Allow any KMS action to Admin",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::<span style="color: #ff0000;">&lt;account-id&gt;</span>:role/<span style="color: #ff0000;">Admin</span>"
      },
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "Allow any KMS action to Managed Flink",
      "Effect": "Allow",
      "Principal": { 
        "Service": [
          "kinesisanalytics.amazonaws.com",
          "infrastructure.kinesisanalytics.amazonaws.com"
        ]
      },
      "Action": [
        "kms:DescribeKey",
        "kms:Decrypt",
        "kms:GenerateDataKey",
        "kms:GenerateDataKeyWithoutPlaintext",
        "kms:CreateGrant"
      ],
      "Resource": "*"
    }
  ]
}</code></pre> 
</div> 
<p>This policy must never be used in an environment containing sensitive data, and especially not in production.</p> 
<h2>Common caveats and pitfalls</h2> 
<p>As discussed earlier, this feature is designed to maximize security and promote best practices such as the principle of least privilege. However, this focus can introduce some corner cases you should be aware of.</p> 
<h3>The CMK must be enabled for the service to encrypt and decrypt snapshots and running state</h3> 
<p>With AWS KMS, you can disable one key at any time. If you disable the CMK while the application is running, it might cause unpredictable failures. For example, an application will not be able to restore a snapshot if the CMK used to encrypt that snapshot has been disabled. For example, if you attempt to roll back an <code>UpdateApplication</code> that changed the CMK, and the previous key has since been disabled, you might not be able to restore from an old snapshot. Similarly, you might not be able to restart the application from an older snapshot if the corresponding CMK is disabled.</p> 
<p>If you encounter these scenarios, the solution is to reenable the required key and retry the operation.</p> 
<h3>The operator requires permissions to all keys involved</h3> 
<p>To perform an action on the application (such as <code>Start</code>, <code>Stop</code>, <code>UpdateApplication</code>, or <code>CreateApplicationSnapshot</code>), the operator must have permissions for all CMKs involved in that operation. AWS owned keys don’t require explicit permission.</p> 
<p>Some operations implicitly involve two CMKs—for example, when switching from one CMK to another, or when switching from a CMK to an AWS owned key by disabling the CMK. In these cases, the operator must have permissions for both keys for the operation to succeed.</p> 
<p>The same rule applies when rolling back an <code>UpdateApplication</code> action that involved multiple CMKs.</p> 
<h3>A new encryption key takes effect only after restart</h3> 
<p>A new encryption key is only used after the application is restarted. This is important when you rotate the key material for a CMK. Rotating the key material in AWS KMS doesn’t require updating the Managed Flink application’s configuration. However, you must restart the application as a separate step after rotating the key. If you don’t restart the application, it will continue to use the old encryption key for its running state and snapshots until the next restart.</p> 
<p>For this reason, it is recommended not to enable automatic key rotation for the CMK. When automatic rotation is enabled, AWS KMS might rotate the key material at any time, but your application will not start using the new key until it is next restarted.</p> 
<h3>CMKs are only supported with Flink runtime 1.20 or later</h3> 
<p>CMKs are only supported when you are using the Flink runtime 1.20 or later. If your application is currently using an older runtime, you should upgrade to Flink 1.20 first. Managed Service for Apache Flink makes it straightforward to upgrade your existing application using the <a href="https://docs.aws.amazon.com/managed-flink/latest/java/how-in-place-version-upgrades.html" rel="noopener noreferrer" target="_blank">in-place version upgrade</a>.</p> 
<h2>Conclusion</h2> 
<p>Managed Service for Apache Flink provides robust security by enabling encryption by default, protecting both the running state and persistently saved state of your applications. For organizations that require full control over their encryption keys (often due to regulatory or internal policy needs), the ability to use a CMK integrated with AWS KMS offers a new level of assurance.</p> 
<p>By using CMKs, you can tailor encryption controls to your specific compliance requirements. However, this flexibility comes with the need for careful planning: the CMK feature is intentionally designed to enforce the principle of least privilege and strong role separation, which can introduce complexity around permissions and operational processes.</p> 
<p>In this post, we reviewed the key steps for enabling CMKs on existing applications, creating new applications with a CMK, and managing key rotation. Each of these processes gives you greater control over your data security but also requires attention to access management and operational best practices.</p> 
<p>To get started with CMKs and for more comprehensive guidance, refer to <a href="https://docs.aws.amazon.com/managed-flink/latest/java/key-management-flink.html" rel="noopener noreferrer" target="_blank">Key management in Amazon Managed Service for Apache Flink</a>.</p> 
<hr /> 
<h3>About the authors</h3> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Lorenzo Nicora" class="aligncenter size-full wp-image-29797" height="160" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2023/09/08/nicorlor.jpeg" width="120" />
  </div> 
  <h3 class="lb-h4">Lorenzo Nicora</h3> 
  <p><a href="https://www.linkedin.com/in/nicus/" rel="noopener" target="_blank">Lorenzo</a> works as Senior Streaming Solution Architect at AWS, helping customers across EMEA. He has been building cloud-centered, data-intensive systems for over 25 years, working across industries both through consultancies and product companies. He has used open-source technologies extensively and contributed to several projects, including Apache Flink, and is the maintainer of the Flink Prometheus connector.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Sofia Zilberman" class="aligncenter size-full wp-image-29797" height="160" src="https://d2908q01vomqb2.cloudfront.net/b6692ea5df920cad691c20319a6fffd7a4a766b8/2025/02/19/sofiez.png" width="120" />
  </div> 
  <h3 class="lb-h4">Sofia Zilberman</h3> 
  <p><a href="https://www.linkedin.com/in/sofie-zilberman/" rel="noopener" target="_blank">Sofia</a> works as a Senior Streaming Solutions Architect at AWS, helping customers design and optimize real-time data pipelines using open-source technologies like Apache Flink, Kafka, and Apache Iceberg. With experience in both streaming and batch data processing, she focuses on making data workflows efficient, observable, and high-performing.</p> 
 </div> 
</footer>
