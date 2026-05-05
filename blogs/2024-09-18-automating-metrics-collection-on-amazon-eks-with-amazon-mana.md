---
title: "Automating metrics collection on Amazon EKS with Amazon Managed Service for Prometheus managed scrapers"
url: "https://aws.amazon.com/blogs/mt/automating-metrics-collection-on-amazon-eks-with-amazon-managed-service-for-prometheus-managed-scrapers/"
date: "Wed, 18 Sep 2024 20:41:29 +0000"
author: "Rodrigue Koffi"
feed_url: "https://aws.amazon.com/blogs/mt/tag/amazon-managed-service-for-prometheus/feed/"
---
<p>Managing and operating monitoring systems for containerized applications can be a significant operational burden for customers such as metrics collection. As container environments scale, customers have to split metric collection across multiple collectors, right-size the collectors to handle peak loads, and continuously manage, patch, secure, and operationalize these collectors. This overhead can detract from an organization’s ability to focus on building and running their applications. To address these challenges, <a href="https://aws.amazon.com/prometheus/">Amazon Managed Service for Prometheus</a> announced a <a href="https://aws.amazon.com/blogs/aws/amazon-managed-service-for-prometheus-collector-provides-agentless-metric-collection-for-amazon-eks/">fully- managed agentless scraper</a> for Prometheus metrics coming from Amazon Elastic Kubernetes Service (Amazon EKS) applications and infrastructure. The fully-managed scraper is a feature that enables customers to collect Prometheus metrics from Amazon EKS environments without installing, patching, updating, managing, or right-sizing any agents in-cluster. This allows customers to offload the “undifferentiated heavy lifting” of self-managing agents to collect Prometheus metrics.</p> 
<h2>Walkthrough</h2> 
<p>In this blog, we’ll walk through how you can use infrastructure-as-code with Terraform or AWS CloudFormation to define an Amazon Managed Service for Prometheus scraper for an existing Amazon EKS cluster. With the support of <a href="https://aws.amazon.com/about-aws/whats-new/2024/05/amazon-managed-service-prometheus-collector-eks-access-management-controls/">Amazon EKS access management controls</a>, we can fully automate Prometheus metrics collection for the Amazon EKS clusters. A high-level architecture with the fully managed scraper looks like the following diagram.</p> 
<p><img alt="Figure 1: High-level architecture for metrics collection with Amazon Managed Service for Prometheus Scraper" class="aligncenter size-full wp-image-56325" height="866" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/09/10/poseidon-figure-1.png" width="1790" /></p> 
<p style="text-align: center;"><em>Figure 1: High-level architecture for metrics collection with Amazon Managed Service for Prometheus scraper</em></p> 
<p>Later, we’ll look into how to update the scraper configuration without disrupting metrics collection and provide insights into how to validate the setup, leveraging <code>usage metrics</code> to follow metrics ingestion into the Managed Prometheus workspace.</p> 
<h3>Prerequisites</h3> 
<ol> 
 <li>An existing Amazon EKS cluster with <a href="https://docs.aws.amazon.com/eks/latest/userguide/cluster-endpoint.html">cluster endpoint access control</a> configured to include private access. It can include private and public access, but must include private access.</li> 
 <li><a href="https://docs.aws.amazon.com/eks/latest/userguide/setting-up-access-entries.html">Amazon EKS authentication mode</a> of the cluster to either the <code>API_AND_CONFIG_MAP</code> or <code>API</code> modes.</li> 
 <li><a href="https://kubernetes.io/docs/tasks/tools/">kubectl</a> version 1.30.2 or later.</li> 
 <li><a href="https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html">AWS Command Line Interface (AWS CLI) version 2.</a></li> 
 <li><a href="https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli">Terraform</a> version 1.8.0 or later.</li> 
</ol> 
<h2>Automate with AWS CloudFormation</h2> 
<p>AWS CloudFormation has released a new resource type <code>AWS::APS::Scraper</code>, to manage the lifecycle of a managed scraper. This resource accepts the following mandatory parameters:</p> 
<ul> 
 <li><a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-aps-scraper-source.html">Source</a>: The source of collected metrics for a scraper. Currently, only a child block <code>EksConfiguration</code> which is an Amazon EKS cluster with cluster endpoint access control at least private, or private and public.</li> 
 <li><a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-aps-scraper-destination.html">Destination</a>: A location for collected metrics. This block can have a child block <code>AmpConfiguration</code> representing the Amazon Resource Name (ARN) of an Amazon Managed Service for Prometheus workspace. Note that the <code>AmpConfiguration</code> block is optional, and if omitted, will trigger the creation of a new workspace by the underlying <a href="https://docs.aws.amazon.com/prometheus/latest/APIReference/API_CreateScraper.html">CreateScraper</a> API.</li> 
 <li><a href="https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-aps-scraper-scrapeconfiguration.html">ScrapeConfiguration</a>: A base-64 encoded Prometheus <a href="https://prometheus.io/docs/prometheus/latest/configuration/configuration/">configuration</a> that specifies which endpoints to collect Prometheus metrics from, the scraping interval (interval of collection), and service discovery (using Kubernetes endpoints for additional metadata).</li> 
</ul> 
<p>In the AWS CloudFormation template, we have a parameter called <code>AMPWorkSpaceAlias</code> , if the value for this is provided as <code>“NONE”</code> then we will create a new AWS Prometheus workspace to store the Amazon EKS cluster metrics. You can alternatively provide the ARN of an existing Amazon Managed Service for Prometheus workspace (in the same region) using the Parameter <code>AMPWorkSpaceArn</code>.</p> 
<p>The first step is to execute few AWS CLI commands to retrieve values like the Security Group ID and Subnet IDs of the Amazon EKS cluster (Note: Please follow the <a href="https://docs.aws.amazon.com/eks/latest/userguide/sec-group-reqs.html">best practices</a>&nbsp;to setup Security of the Amazon EKS cluster), which are required for the creation of the Scraper. The output from the below commands will be used as value for the Input Parameters defined in the AWS CloudFormation stack in the next step. Replace <code>&lt;EKS_CLUSTER&gt;</code> with your Amazon EKS cluster name.</p> 
<pre><code class="lang-bash"># selecting one security group associated to the cluster's VPC
aws eks describe-cluster —name &lt;EKS_CLUSTER&gt; | jq .cluster.resourcesVpcConfig.securityGroupIds[0]

# selecting cluster's subnets
aws eks describe-cluster —name &lt;EKS_CLUSTER&gt; | jq .cluster.resourcesVpcConfig.subnetIds[0]
aws eks describe-cluster —name &lt;EKS_CLUSTER&gt; | jq .cluster.resourcesVpcConfig.subnetIds[1]</code></pre> 
<p>Now let’s build the AWS CloudFormation template, running the commands below to create the Scraper. Please replace <code>WORKSPACE_ARN</code>, <code>EKS_CLUSTER_ARN</code>, <code>EKS_SECURITY_GROUP_ID</code>, <code>SUBNET_ID_1</code>, <code>SUBNET_ID_2</code> with their respective values retrieved earlier.</p> 
<pre><code class="lang-bash">git clone https://github.com/aws-samples/containers-blog-maelstrom.git
cd containers-blog-maelstrom/amp-scraper-automation-blog/cloudformation
aws cloudformation create-stack --stack-name AMPScraper \
    --template-body file://scraper.yaml \
    --parameters AMPWorkSpaceArn=&lt;WORKSPACE_ARN&gt; \
    ClusterArn=&lt;EKS_CLUSTER_ARN&gt; \
    SecurityGroupId=&lt;EKS_SECURITY_GROUP_ID&gt; \
    SubnetId1=&lt;SUBNET_ID_1&gt; \
    SubnetId2=&lt;SUBNET_ID_2&gt;</code></pre> 
<p>After running these commands, the managed scraper creation will leverage <a href="https://docs.aws.amazon.com/eks/latest/userguide/access-entries.html">Amazon EKS access entries</a> to automatically provide, Amazon Managed Service for Prometheus scraper, access to your cluster. The AWS CloudFormation stack will take a few minutes to complete. In the next sections of the blog, we will confirm the resources are created as expected, either via the cli or the console.</p> 
<h2>Automate with Terraform</h2> 
<p>Now let’s see how to leverage Terraform for the end-to-end setup.</p> 
<p>We will be creating an Amazon Managed Service for Prometheus fully managed scraper using the Terraform resource <code>aws_prometheus_scraper</code>. Make sure to complete the pre-requisites for the managed scraper creation to leverage Amazon EKS access entries to automatically provide, Amazon Managed Service for Prometheus scraper, access to your cluster.</p> 
<p>Run these commands below and replace <code>EKS_CLUSTER</code> with your Amazon EKS cluster name.</p> 
<pre><code class="lang-bash">git clone https://github.com/aws-samples/containers-blog-maelstrom.git
cd containers-blog-maelstrom/amp-scraper-automation-blog/terraform
terraform init
terraform apply -var eks_cluster_name="EKS_CLUSTER"</code></pre> 
<h2>Validation</h2> 
<p>In the next sections, we will confirm that our managed scraper has been created, associated to the cluster and effectively collecting metrics.</p> 
<h3>Using AWS CLI</h3> 
<p>The <code>list-scrapers</code> CLI action allows to retrieve all the scrapers created. You can provide a filter to narrow down your search. In our example below, we filter on the alias <code>amp-scraper-automation</code> used in Terraform or AWS CloudFormation.</p> 
<pre><code class="lang-bash">aws amp list-scrapers --filter alias=amp-scraper-automation</code></pre> 
<p><img alt="Figure 2 - View managed scraper using AWS CLI" class="aligncenter size-full wp-image-56324" height="1073" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/09/10/poseidon-figure-2.png" width="2259" /></p> 
<p style="text-align: center;"><em>Figure 2 – View managed scraper using AWS CLI</em></p> 
<h3>Using AWS Management Console</h3> 
<p>Login to the AWS account and Region where your Amazon EKS cluster is created. Select the cluster, click on the <code>Observability</code> tab and you should see the <code>scraper</code> created (as shown in the below screenshot).</p> 
<p><img alt="Figure 3 - View managed scraper in the Amazon EKS console" class="aligncenter size-full wp-image-56323" height="902" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/09/10/poseidon-figure-3.png" width="2140" /></p> 
<p style="text-align: center;"><em>Figure 3 – View managed scraper in the Amazon EKS console</em></p> 
<h3>Amazon CloudWatch Usage metrics</h3> 
<p>Amazon Managed Service for Prometheus will publish its <a href="https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-CW-usage-metrics.html">usage metrics</a> into <a href="https://aws.amazon.com/cloudwatch/">Amazon CloudWatch</a>. This allows you to<br /> immediately have insights into the AWS EKS workspace utilization. You can set up Amazon CloudWatch alarms to track some of those metrics, depending on your use case. If you followed the steps above, you should be able to view these metrics in the Amazon CloudWatch console.</p> 
<p>Looking at the Amazon CloudWatch Usage metric namespace, we select the <code>IngestionRate</code> and <code>ActiveSeries</code> metrics to validate and monitor the usage against <a href="https://docs.aws.amazon.com/prometheus/latest/userguide/AMP_quotas.html">service quotas</a>, as shown in the following figure.</p> 
<p><img alt="Figure 4 - Viewing Usage metrics in Amazon CloudWatch" class="aligncenter size-full wp-image-56322" height="520" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/09/10/poseidon-figure-4.png" width="2329" /></p> 
<p style="text-align: center;"><em>Figure 4 – Viewing Usage metrics in Amazon CloudWatch</em></p> 
<p>Let’s see some examples of setting-up Amazon CloudWatch Alarms for these Prometheus ingested metrics:</p> 
<ul> 
 <li><code>ActiveSeries</code> – The quota on active series per workspace will be automatically adjusted to a certain point (as mentioned in the Service Quota page). To grow above, we can setup an Amazon CloudWatch Alarm to monitor its usage. For example, when ActiveSeries is above 10M, we receive an Alarm so that we can request a quota increase.</li> 
 <li><code>IngestionRate</code> – We could use a DIFF and/or RATE <a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/using-metric-math.html">metric math function</a> to validate if there is any spike in the Ingestion that could be coming from a misconfiguration or if some teams are suddenly ingesting too many metrics.</li> 
</ul> 
<p>Amazon CloudWatch will also create an Automatic Dashboard for these metrics ingested from Amazon Managed service for Prometheus.</p> 
<p>Something important to note is that the managed scraper does not consume any resource in the source Amazon EKS cluster. If you list all the pods running in the cluster with the following command, try and see if you can spot any scraper!</p> 
<pre><code class="lang-bash">kubectl get pods --all</code></pre> 
<h3>High availability metrics data to avoid duplicate metrics</h3> 
<p>Now let’s see how to update the scraper configuration without disrupting the metrics collection, for this we will configure high-availability data with Amazon Managed Service for Prometheus. We need to add an <code>external_labels</code> under the <code>global</code> section, this can be any <code>key:value</code> pair. Here we are adding a label called <code>source</code> with a value <code>reduce_metrics</code>, and we have also reduced the metrics from the configuration, by just keeping the <code>pod_exporter</code> for this example.</p> 
<p><img alt="Figure 5 - High-level architecture for HA metrics collection using scrapers" class="aligncenter wp-image-56321 size-full" height="884" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/09/10/poseidon-figure-5.png" width="1788" /></p> 
<p style="text-align: center;"><em>Figure 5 – High-level architecture for HA metrics collection using scrapers</em></p> 
<p>Using Terraform as an example, we can add a new <code>aws_prometheus_scraper</code> resource block in the same file. In the snippet below, showing only the difference with the first scraper resource, we are using a smaller scrape configuration to collect less metrics. Note that we are adding <code>external_labels</code> under the global section of the scrape configuration.</p> 
<pre><code class="lang-yaml">resource "aws_prometheus_scraper" "reduce_samples" {
  # ... omitted for brevity 
  scrape_configuration = &lt;&lt;EOT
global:
  scrape_interval: 30s
  external_labels: 
    source: reduce_metrics
scrape_configs:
   # cluster config
  # pod metrics
  - job_name: pod_exporter
    kubernetes_sd_configs:
      - role: pod
EOT

  # ... omitted for brevity 
}</code></pre> 
<p>Append the snippet above to your Terraform file and execute the commands <code>terraform</code><br /> <code>plan</code> and then <code>terraform apply</code>, and you should see a new Scraper being created.</p> 
<p><img alt="Figure 6 - Viewing creation of second scraper from the AWS console" class="aligncenter wp-image-56320 size-full" height="996" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/09/10/poseidon-figure-6.png" width="2150" /></p> 
<p style="text-align: center;"><em>Figure 6 – Viewing creation of second scraper from the AWS console</em></p> 
<p>Once the new scraper is Active, you can delete the old scraper by removing the below resource block for <code>aws_prometheus_scraper</code>.</p> 
<pre><code class="lang-bash">- resource "aws_prometheus_scraper" "this" {
- ...
- }
</code></pre> 
<p>Again, execute the commands <code>terraform plan</code> and then <code>terraform</code><code>apply</code> to apply the changes.</p> 
<h3>Visualize in Grafana</h3> 
<p>Using the <code>Explore</code> feature in Grafana, we can create our own queries by selecting desired metrics and filters. We can add them to an existing dashboard or create a new. We will use Explore to query our Amazon Managed Service for Prometheus workspace. Follow the <a href="https://docs.aws.amazon.com/grafana/latest/userguide/prometheus-data-source.html">AWS documentation</a> to setup Amazon Managed Grafana.</p> 
<p>We can see that the <code>“reduce_metrics” external_label</code> we added to our scraper config is now available under Explore → Label filters and can be used to create visualizations.</p> 
<p><img alt="Figure 7 - Validating the external label added with the second HA scraper" class="aligncenter size-full wp-image-56319" height="1548" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/09/10/poseidon-figure-7.png" width="2816" /></p> 
<p style="text-align: center;"><em>Figure 7 – Validating the external label added with the second HA scraper</em></p> 
<p>We can also confirm that the metrics are not duplicated while we had two managed scrapers running simultaneously.</p> 
<p><img alt="Figure 8 - Validating HA and de-duplication of metrics" class="aligncenter size-full wp-image-56318" height="948" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/09/10/poseidon-figure-8.png" width="2856" /></p> 
<p style="text-align: center;"><em> Figure 8 – Validating HA and de-duplication of metrics</em></p> 
<h2>Cleaning up</h2> 
<p>To delete the resources created in this post, run these commands, depending on the path you chose to avoid continuing incurring charges.</p> 
<p><strong>CloudFormation:</strong></p> 
<p><code>aws cloudformation delete-stack --stack-name AMPScraper</code></p> 
<p><strong>Terraform:</strong></p> 
<p><code>terraform destroy</code></p> 
<h2>Conclusion</h2> 
<p>In this blog, we’ve walked through how you can create an Amazon Managed Service for Prometheus scraper through Infrastructure as code tools such as Terraform and AWS CloudFormation. With the integration between the managed scraper and Amazon EKS access management controls, you can now programmatically create scrapers, and associate them with your Amazon EKS clusters with simple, repeatable and predictable deployments. By using the managed scraper, you can reduce your operational load and have AWS scale your ingestion to match your traffic. We’ve also shown how to update the managed scraper without disrupting your metrics collection. Finally, we have seen how to leverage CloudWatch metrics to follow the ingestion of an Amazon Managed Service for Prometheus workspace.</p> 
<ul> 
 <li>To go further with monitoring Amazon EKS clusters, check out our <a href="https://docs.aws.amazon.com/grafana/latest/userguide/solution-eks.html">end-to-end solution</a> with opinionated metrics collection, dashboards and alarms, with infrastructure-as-code.</li> 
 <li>Check out <a href="https://catalog.workshops.aws/observability/en-US">One Observability Workshop</a> aimed at providing a hands-on experience for you on the wide variety of toolsets AWS offers to setup monitoring and observability on your applications.</li> 
 <li>Refer to <a href="https://aws-observability.github.io/observability-best-practices/">AWS Observability best practices</a> to learn more about prescriptive guidance and recommendations with implementation examples.</li> 
</ul> 
<footer></footer>
