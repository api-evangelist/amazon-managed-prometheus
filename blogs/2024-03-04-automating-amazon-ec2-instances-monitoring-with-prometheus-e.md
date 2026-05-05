---
title: "Automating Amazon EC2 Instances Monitoring with Prometheus EC2 Service Discovery and AWS Distro for OpenTelemetry"
url: "https://aws.amazon.com/blogs/mt/automating-amazon-ec2-instance-monitoring-with-prometheus-ec2-service-discovery-and-aws-distro-for-opentelemetry/"
date: "Mon, 04 Mar 2024 13:26:24 +0000"
author: "Jay Joshi"
feed_url: "https://aws.amazon.com/blogs/mt/tag/amazon-managed-service-for-prometheus/feed/"
---
<p>Traditionally, scraping application Prometheus metrics required manual updates to a configuration file, posing challenges in dynamic AWS environments where Amazon EC2 instances are frequently created or terminated. This not only proves time consuming but also introduces the risk of configuration errors, lacking the agility necessary in dynamic environments.</p> 
<p>In this blog post, we will demonstrate how Prometheus service discovery, particularly <a href="https://prometheus.io/docs/prometheus/latest/configuration/configuration/#ec2_sd_config">EC2 service discovery</a>, can help overcome these challenges providing the following benefits:</p> 
<ul> 
 <li>Automatic target discovery</li> 
 <li>Reduced manual effort and enhanced agility</li> 
 <li>Minimized configuration errors</li> 
</ul> 
<p>We will showcase how to configure <a href="https://aws-otel.github.io/docs/getting-started/collector">AWS Distro for OpenTelemetry (ADOT) collector</a> to perform EC2 service discovery in order to dynamically identify the EC2 targets for scraping Prometheus metrics. Subsequently, we are going to simulate a dynamic environment to showcase how EC2 service discovery automatically updates the list of targets to be scraped. We will collect the Prometheus metrics using <a href="https://aws.amazon.com/prometheus/">Amazon Managed Service for Prometheus</a> workspace and visualize them using <a href="https://aws.amazon.com/grafana/">Amazon Managed Grafana</a>.</p> 
<h2><strong><strong>Solution Overview</strong></strong></h2> 
<p>To showcase the dynamic discovery of EC2 instance targets using EC2 service discovery, we are going to provision the following resources through <a href="https://aws.amazon.com/cloudformation/">AWS CloudFormation</a>:</p> 
<ul> 
 <li>AWS Distro for OpenTelemetry (ADOT) collector running on EC2 instance with name <code>ADOT_COLLECTOR</code> to scrape Prometheus metrics.</li> 
 <li>Two Amazon EC2 instances with name <code>APP_SERVER</code> launched by an AWS AutoScaling Group (ASG) named <code>ApplicationASG</code><strong>.</strong> They will be configured to run <a href="https://github.com/prometheus/node_exporter"><code>node_exporter</code></a> to expose OS level Prometheus metrics.</li> 
 <li>The ADOT collector is configured to dynamically identify these targets using EC2 service discovery and filter them based on <strong><code>tag-key=service_name</code> </strong>and <strong><code>tag-value=node_exporter</code></strong>.</li> 
 <li>An Amazon Managed Service for Prometheus and Amazon Managed Grafana workspace.</li> 
</ul> 
<p><img alt="EC2 Service Discovery Archiecture with ADOT collector" class="alignnone wp-image-49750 size-full" height="2856" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/03/03/ec2_sd_diagram_archiecture-Simple-Scrape.drawio.png" width="3872" /><em>Figure 1: Solution Architecture</em></p> 
<h2><strong><strong>Prerequisites</strong></strong></h2> 
<ol> 
 <li>Before starting, make sure you have <a href="https://docs.aws.amazon.com/cloudshell/latest/userguide/welcome.html">AWS CloudShell</a>, a browser-based shell setup into your AWS account and region to run the commands described in this blog post.</li> 
 <li>(Optional) We will be configuring user access through <a href="https://aws.amazon.com/single-sign-on/">AWS IAM Identity Center</a> for Amazon Managed Grafana workspace. Make sure you have enabled <a href="https://docs.aws.amazon.com/singlesignon/latest/userguide/regions.html">IAM Identity Center</a> in your AWS account.</li> 
</ol> 
<h2><strong><strong>Solution Walkthrough</strong></strong></h2> 
<p>To deploy the architecture shown in <strong>Figure 1</strong>, please follow the below steps:</p> 
<ol> 
 <li>From the AWS CloudShell command line interface, enter the below commands to clone the sample project from the <code>aws-samples</code> <a href="https://github.com/aws-samples/amazon-ec2-dynamic-monitoring-with-prometheus-service-discovery.git">GitHub repository</a>. <pre><code class="lang-bash">git clone https://github.com/aws-samples/amazon-ec2-dynamic-monitoring-with-prometheus-service-discovery.git 
cd amazon-ec2-dynamic-monitoring-with-prometheus-service-discovery/templates</code></pre> </li> 
 <li>Next, to provision the resources, enter the following command. Replace the <code>&lt;aws-region&gt;</code> with your AWS Region name. <pre><code class="lang-bash">AWS_REGION=&lt;aws-region&gt;
aws cloudformation create-stack --stack-name adot-ec2-service-discovery-demo --template-body file://adot_ec2_service_discovery_cfn.yml --capabilities CAPABILITY_IAM --region $AWS_REGION</code></pre> </li> 
</ol> 
<h2><strong><strong>Setting up Amazon Managed Grafana Workspace</strong></strong></h2> 
<p>A managed Grafana workspace has been already created using AWS CloudFormation. Next you need to set up the following two configurations on this workspace:</p> 
<ul> 
 <li>Amazon Managed Grafana lets you to configure user access through <a href="https://aws.amazon.com/single-sign-on/">AWS IAM Identity Center</a> or other <a href="https://aws.amazon.com/blogs/mt/amazon-managed-grafana-supports-direct-saml-integration-with-identity-providers/">SAML based Identity Providers (IdP)</a>. In this post, we’re using the AWS IAM Identity Center option with Amazon Managed Grafana. To set up Authentication and Authorization, follow the instructions in the <a href="https://docs.aws.amazon.com/grafana/latest/userguide/AMG-manage-users-and-groups-AMG.html">Amazon Managed Grafana User Guide</a> for enabling AWS IAM Identity Center.</li> 
</ul> 
<p><img alt="Console screenshot of Amazon Managed Grafana user access using AWS SSO." class="alignnone size-full wp-image-49619" height="758" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/02/28/grafana-sso.png" width="2736" /><em>Figure 2: Example of Amazon Managed Grafana user access using AWS SSO.</em></p> 
<ul> 
 <li>Further, follow these <a href="https://docs.aws.amazon.com/grafana/latest/userguide/AMP-adding-AWS-config.html">steps</a> to configure Amazon Managed Service for Prometheus as a data source for this Amazon Managed Grafana workspace.</li> 
</ul> 
<p><em><img alt="Console screenshot for Configuring Amazon Managed Service for Prometheus as a data source for Amazon Managed Grafana" class="alignnone size-full wp-image-49621" height="1486" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/02/28/amg-datasource.png" width="2874" />Figure 3: Configuring Amazon Managed Prometheus as data source for Amazon Managed Grafana</em></p> 
<h2><strong><strong>Visualizing Prometheus Metrics with Amazon Managed Grafana</strong></strong></h2> 
<p>Now, let’s visualize the Prometheus metrics that have been pushed by the ADOT collector to the Amazon Managed Service for Prometheus workspace.</p> 
<p>Navigate to Amazon Managed Grafana workspace from your AWS Management Console, choose the Workspace URL to sign in to your Grafana dashboard. As demonstrated in <strong>Figure 4</strong>, we are visualizing the Prometheus metric <code>node_cpu_seconds_total</code> for all the EC2 target instances that were dynamically discovered by the ADOT collector agent using EC2 service discovery.</p> 
<p><strong><img alt="Visualizing Prometheus metrics of dynamically scrapped targets on Managed Grafana console" class="alignnone size-full wp-image-49622" height="920" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/02/28/grafana-visualize.png" width="1893" /></strong></p> 
<p><em>Figure 4: Visualizing Prometheus metrics of dynamically scrapped targets</em></p> 
<p>Additionally, you can visualize Prometheus metrics for individual EC2 instance targets by utilizing the <code>instance_id</code> label, as shown in <strong>Figure 5</strong>.</p> 
<p><strong><img alt="Visualizing Prometheus metrics of specific scrapped target on Grafana console" class="alignnone size-full wp-image-49625" height="1956" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/02/28/filter_instanceid_grafana.png" width="3839" /></strong><em>Figure 5: Visualizing Prometheus metrics of specific scrapped target</em></p> 
<h2><strong><strong>Simulating Dynamic EC2 Environment</strong></strong></h2> 
<p>To simulate a dynamic environment, we will increase the <strong>“Desired capacity”</strong> of the <code>ApplicationASG</code> Auto Scaling Group. Currently, this ASG is configured with a minimum size of 2, a maximum size of 4, and a desired capacity of 2. We will adjust the <code>Desired capacity </code>value from <strong>2</strong> to <strong>4.</strong> Please follow the below steps to change this parameter:</p> 
<h4><strong><strong>Steps:</strong></strong></h4> 
<ol> 
 <li>Navigate to <a href="https://console.aws.amazon.com/cloudshell/home#">AWS CloudShell console</a>.</li> 
 <li>Run the following AWS CLI command in the terminal: <pre><code class="lang-bash">ASG_NAME=$(aws cloudformation describe-stacks --stack-name adot-ec2-service-discovery-demo --region $AWS_REGION --query 'Stacks[0].Outputs[?OutputKey==`ASG`].OutputValue' --output text)
echo $ASG_NAME 
aws autoscaling set-desired-capacity --auto-scaling-group-name $ASG_NAME --desired-capacity 4 --honor-cooldown --region $AWS_REGION</code></pre> </li> 
</ol> 
<p>Wait 2-5 minutes for the ADOT collector to identify the new EC2 targets launched by the ASG service. Then, navigate to your Amazon Managed Grafana console to visualize the associated Prometheus metrics for these targets (see Figure 6).</p> 
<p><strong><img alt="Visualizing Prometheus metrics of newly launched targets on grafana console" class="alignnone size-full wp-image-49627" height="837" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/02/28/dynamic-targets-discovery.png" width="1911" /></strong><em>Figure 6: Visualizing Prometheus metrics of newly launched targets</em></p> 
<p>This showcases how ADOT collector leverages EC2 service discovery to identify newly added EC2 instances during scale-out activities in the Auto Scaling Group (ASG) and seamlessly collects Prometheus metrics from the newly identified targets, facilitating real-time monitoring and scalability within dynamic environments.</p> 
<p>Let’s delve into how the ADOT collector manages to automatically identify these newly launched targets:</p> 
<ul> 
 <li>The ADOT collector initiates a <code>DescribeInstances</code> API call, specifying <code>filter</code> parameters to search for instances tagged with <code>service_name</code> as the key and <code>node_exporter</code> as the value.</li> 
 <li>The EC2 API responds with a filtered list of instances that meet the specified criteria. This updated list now includes the two recently launched instances from the ASG. This list is automatically refreshed based on the <code>refresh_interval</code> parameter.</li> 
 <li>The filtered targets will then be scraped by the ADOT collector in order to collect Prometheus metrics.</li> 
 <li>Prometheus metrics are retrieved from the targets and substantially pushed to desired destination e.g. Amazon Service for Managed Prometheus in this scenario.</li> 
 <li>Amazon Managed Grafana then queries Prometheus metrics from Amazon Managed Service for Prometheus.</li> 
</ul> 
<p><img alt="EC2 Service Discovery Flow Diagram" class="alignnone size-full wp-image-49610" height="1546" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/02/28/ec2_api_sd_flow_latest.png" width="2294" /> <em>Figure 7: Flow diagram of how ADOT collector performs EC2 service discovery</em></p> 
<h2><strong>Design Considerations</strong></h2> 
<p>Here are some key design aspects you should consider while configuring EC2 service discovery with the ADOT collector on Amazon EC2.</p> 
<h4><strong><strong>1. IAM Role Permissions</strong></strong></h4> 
<p>When deploying the ADOT collector in conjunction with EC2 service discovery, make sure EC2 IAM Role must be equipped with <code>ec2:DescribeInstances</code> and <code>ec2:DescribeAvailabilityZones</code> permissions.</p> 
<h4><strong><strong>2. DescribeInstances API Requests Limit</strong></strong></h4> 
<p>By default, the ADOT collector will refresh the list of EC2 instances every 60s by making <code>DescribeInstances</code> API. You can configure the <a href="https://prometheus.io/docs/prometheus/latest/configuration/configuration/#ec2_sd_config"><strong><code>refresh_interval</code></strong></a> option to control how frequently ADOT collector makes the API requests in order to update this list. An example of such configuration can be found below snippet:</p> 
<pre><code class="lang-yaml"># EC2 service discovery with refresh interval as 5 minutes
  - job_name: 'node_exporter'
    ec2_sd_configs:
    - region: eu-west-1
    - refresh_interval: 5m </code></pre> 
<p>Refer <a href="https://docs.aws.amazon.com/AWSEC2/latest/APIReference/throttling.html#throttling-how">Request throttling for the Amazon EC2 API</a> for more information.</p> 
<p><strong><strong>3. Configuring EC2 Security Groups</strong></strong></p> 
<p>EC2 Service Discovery uses the EC2 instance private IP Address by default to scrape Prometheus metric. In order for ADOT collector to successfully scrape EC2 instances in VPC make sure you allow <code>ingress</code> traffic on port to scrape metrics under security group associated with your instances. For instance, if your application exposes Prometheus metrics via TCP port 9100, make sure to allow ingress traffic specifically on this port within the security group settings.</p> 
<h4><strong><strong>4. Tagging Strategies to Discover EC2 Instances</strong></strong></h4> 
<p>Tagging is a crucial aspect of effectively utilizing Prometheus EC2 service discovery. Employ essential metadata tags like <code>Application</code> or <code>Service Name</code>, <code>Environment Name</code>, and <code>Role</code> or <code>Function</code> to streamline grouping and identification of instances. Additionally, implement hierarchical tags, such as <code>tier</code> or <code>cluster</code> to represent relationships and dependencies, facilitating organized monitoring.</p> 
<p>These best practices empower selective and targeted discovery, ensuring efficient monitoring of EC2 instances in dynamic AWS environments. Further insights can be found under the <a href="https://docs.aws.amazon.com/whitepapers/latest/tagging-best-practices/tagging-best-practices.html">Tagging Best Practices Whitepaper</a>.</p> 
<h4><strong>5. Scaling ADOT Collector</strong></h4> 
<p>Below are some ADOT collector scaling strategies running on EC2 instances while scraping a large number of targets:</p> 
<ul> 
 <li><strong><strong>Vertical Scaling:</strong></strong> Initiate the scaling process by vertically expanding your ADOT Collector instance. This involves allocating more CPU and memory resources. You can accomplish this by modifying the EC2 instance type on which the ADOT collector runs.</li> 
 <li><strong><strong>Sharding by Availability Zones (AZ):</strong></strong> In cases where you are scraping metrics from a vast array of EC2 instances spread across multiple Availability Zones (AZ) within a VPC, consider sharding the ADOT collector instance per AZ. This approach evenly distributes the workload across multiple ADOT collector instances. Below snippet is an example ADOT configuration to achieve this: <pre><code class="lang-yaml"># ADOT Collector configuration to scrape targets from specific Availability Zone "ap-south-1a"
---
ec2_sd_configs:
  - region: ap-south-1
    port: 9100
    filters:
      - name: __meta_ec2_availability_zone
        values:
          - ap-south-1a
relabel_configs:
  - source_labels:
      - __meta_ec2_instance_id
    target_label: instance_id</code></pre> </li> 
 <li><strong>Sharding by Metrics Type:</strong> Another sharding approach is based on the type of metrics you want to collect. For example, if you are running <code>node_exporter</code> to gather infrastructure-level metrics and <code>jmx_exporter</code> to collect application-level metrics, you can distribute the collection of these metrics using two ADOT collector instances. Likewise, you can shard them based on the environment or application. Here’s a snippet of ADOT configuration to achieve this: <pre><code class="lang-yaml"># Scraping targets running jmx exporter by filtering using tag key "application" and value "JMX"
---
ec2_sd_configs:
  - region: ap-south-1
    port: 9999
    filters:
      - name: tag:application
        values:
          - JMX
relabel_configs:
  - source_labels:
      - __meta_ec2_instance_id
    target_label: instance_id</code></pre> </li> 
</ul> 
<h2><strong><strong>Cleaning up</strong></strong></h2> 
<p>To decommission all the resources deployed during walkthrough, navigate to AWS CloudShell command line interface and run the below command.</p> 
<pre><code class="lang-bash">aws cloudformation delete-stack --stack-name adot-ec2-service-discovery-demo --region $AWS_REGION </code></pre> 
<h2><strong><strong>Conclusion</strong></strong></h2> 
<p>In this blog post, we demonstrated how you can use EC2 service discovery with AWS Distro for OpenTelemetry (ADOT) collector in order to automatically identify targets for scraping Prometheus metrics from dynamic EC2 environments. This leads to significant reduction of time spent to manually maintaining list of targets and also mitigates the risk of configuration errors.</p> 
<p>We also highlighted key design considerations aimed at enhancing operational efficiency and ensuring a more reliable monitoring process while using EC2 service discovery with ADOT collector. As a next step, we encourage you to try and customize this solution for your specific use cases in managing Prometheus metric scraping with ADOT collector in dynamic EC2 environments.</p> 
<p>To learn more about AWS Observability services, please check the below resources:</p> 
<ul> 
 <li><a href="https://catalog.workshops.aws/observability/en-US/intro">Hands-on experience with AWS Observability Workshop</a></li> 
 <li><a href="https://aws-observability.github.io/observability-best-practices/">AWS Observability Best Practices Guide</a></li> 
 <li><a href="https://aws-observability.github.io/cdk-aws-observability-accelerator/">AWS Observability Accelerator for CDK</a></li> 
 <li><a href="https://aws-observability.github.io/terraform-aws-observability-accelerator/">AWS Observability Accelerator for Terraform</a></li> 
 <li><a href="https://explore.skillbuilder.aws/learn/course/external/view/elearning/14688/aws-observability">Free course on AWS Skill Builder – Observability</a></li> 
</ul>
