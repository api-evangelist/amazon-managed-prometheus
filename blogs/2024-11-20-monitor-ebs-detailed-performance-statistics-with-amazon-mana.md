---
title: "Monitor EBS Detailed Performance Statistics with Amazon Managed Service for Prometheus"
url: "https://aws.amazon.com/blogs/mt/monitor-ebs-detailed-performance-statistics-with-amazon-managed-service-for-prometheus/"
date: "Wed, 20 Nov 2024 15:22:48 +0000"
author: "Mike George"
feed_url: "https://aws.amazon.com/blogs/mt/tag/amazon-managed-service-for-prometheus/feed/"
---
<p>Today we are excited to announce that you can now easily ingest <a href="https://aws.amazon.com/ebs/">Amazon EBS</a> detailed performance statistics from your <a href="https://aws.amazon.com/eks/">Amazon Elastic Kubernetes Service (Amazon EKS)</a> workloads into an <a href="https://aws.amazon.com/prometheus/">Amazon Managed Service for Prometheus</a> workspace. We recently announced the availability of <a href="https://docs.aws.amazon.com/ebs/latest/userguide/nvme-detailed-performance-stats.html">EBS detailed performance statistics</a>, which gives you real-time visibility into the performance of your EBS storage volumes. Prior to this release, customers who needed granular volume-level observability had to use a patchwork of system-level tools. EBS detailed performance statistics gives you access to 11 high-performance metrics at sub-minute granularity. You can use these statistics to better understand the health and performance of your Kubernetes storage. In this blog post, we will demonstrate how you can enable detailed performance statistics on EBS volumes in your EKS clusters and send this telemetry data to your Prometheus workspace.</p> 
<p><strong>Getting started</strong></p> 
<p>There are several pre-requisites needed for collecting the new EBS detailed performance statistics.</p> 
<ol> 
 <li>Install the <a href="https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html">AWS CLI</a> and <a href="https://eksctl.io/installation/">eksctl</a> command line tools.</li> 
 <li>You must have an Amazon EKS cluster. If you don’t have an EKS cluster, you can follow this guide to <a href="https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html">get started</a>. You can also create a cluster via the CLI as follows (where <code>&lt;clusterName&gt;</code> is the name of the cluster you want to create): 
  <div class="hide-language"> 
   <pre><code class="lang-bash">eksctl create cluster --name &lt;clusterName&gt;</code></pre> 
  </div> </li> 
 <li>The cluster <a href="https://eksctl.io/usage/iamserviceaccounts/">must have OIDC enabled</a> so that you can map IAM roles to Kubernetes service accounts. The easiest way to do this is via <a href="https://eksctl.io/">eksctl</a>. You can run the following command on an EKS cluster to enable the IAM OIDC Provider, where <code>&lt;clusterName&gt;</code> is the name of your EKS cluster: 
  <div class="hide-language"> 
   <pre><code class="lang-bash">eksctl utils associate-iam-oidc-provider --cluster=&lt;clusterName&gt;</code></pre> 
  </div> </li> 
 <li><a href="https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html#csi-iam-role">Create an IAM role</a> so the CSI driver for Amazon EBS has the correct permissions for Kubernetes to access storage volumes. You can do this by running the following command, where <code>&lt;clusterName&gt;</code> is the name of the EKS cluster: 
  <div class="hide-language"> 
   <pre><code class="lang-bash">eksctl create iamserviceaccount \
	--name ebs-csi-controller-sa \
	--namespace kube-system \
	--cluster &lt;clusterName&gt; \
	--role-name AmazonEKS_EBS_CSI_DriverRole \
	--role-only \
	--attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
	--approve</code></pre> 
  </div> </li> 
 <li>Install the Amazon EBS CSI Driver. This can be done either via the <a href="https://docs.aws.amazon.com/eks/latest/userguide/creating-an-add-on.html#_create_add_on_console">EKS console or the AWS CLI</a>. You can use the following command, where <code>&lt;clusterName&gt;</code> is the name of the EKS cluster and <code>&lt;roleArn&gt;</code> is the ARN of the <code>AmazonEKS_EBS_CSI_DriverRole</code> created in the previous step. 
  <div class="hide-language"> 
   <pre><code class="lang-bash">eksctl create addon --name aws-ebs-csi-driver --cluster &lt;clusterName&gt; --service-account-role-arn &lt;roleArn&gt;</code></pre> 
  </div> </li> 
 <li>Ensure you have an Amazon Managed Service for Prometheus workspace. If you don’t already have a workspace, you can follow this guide to <a href="https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-getting-started.html">get started</a>. You can also create a cluster via the CLI as follows (where <code>&lt;clusterWorkspaceName&gt;</code> is the name of the workspace you want to create): 
  <div class="hide-language"> 
   <pre><code class="lang-bash">aws amp create-workspace –-alias &lt;clusterWorkspaceName&gt;</code></pre> 
  </div> </li> 
 <li>Ensure you have an <a href="https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-collector-how-to.html">AWS managed collector</a> configured as part of your EKS cluster. You can create a scraper as part of your EKS cluster creation, or you can create a scraper via the AWS API or AWS CLI for existing clusters. The blog <a href="https://aws.amazon.com/blogs/aws/amazon-managed-service-for-prometheus-collector-provides-agentless-metric-collection-for-amazon-eks/">Amazon Managed Service for Prometheus collector provides agentless metric collection for Amazon EKS</a> provides more details on how to configure the agentless metric collector for new and existing clusters. You can view existing and create new scrapers from the <strong>Observability</strong> tab of the EKS cluster (see figure 1).</li> 
</ol> 
<p><img alt="The EKS cluster information page, where the Observability tab is selected. In the details, an Agentless Prometheus scraper is in the creating stage." class="alignnone size-full wp-image-57960" height="1011" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/11/06/figure1.png" width="2062" />Figure 1: Agentless Prometheus scraper in the EKS cluster</p> 
<p><strong>Enabling EBS detailed performance statistics in EKS</strong></p> 
<p>Once you have these pre-requisites configured, enabling collection of the EBS detailed performance statistics consists of <a href="https://docs.aws.amazon.com/eks/latest/userguide/updating-an-add-on.html">updating the Amazon EBS CSI Driver</a> and enabling metrics for the node plugin.</p> 
<p>First, check your current add-on version:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">eksctl get addon --cluster &lt;clusterName&gt;</code></pre> 
</div> 
<p>The command should show that v1.37.0 or later is available for the aws-ebs-csi-driver add-on. To enable metrics collection, you will need to update the add-on with advanced configuration. Create a file named <code>values.yaml</code> with the following content:</p> 
<div class="hide-language"> 
 <pre><code class="lang-yaml">node:
  enableMetrics: true</code></pre> 
</div> 
<p>Then update the add-on using the AWS CLI:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">aws eks update-addon \
--cluster-name &lt;clusterName&gt; \
--addon-name aws-ebs-csi-driver \
--resolve-conflicts OVERWRITE \
--configuration-values file://values.yaml</code></pre> 
</div> 
<p>The key configuration here is <code>node.enableMetrics: true</code>, which enables the collection of the detailed performance statistics. For more information about EKS add-on advanced configuration options, see the <a href="https://aws.amazon.com/blogs/containers/amazon-eks-add-ons-advanced-configuration/">Amazon EKS Add-ons: Advanced configuration</a> blog.</p> 
<p>After updating the add-on with metrics enabled, the EBS detailed performance statistics will be automatically scraped and sent to your Prometheus workspace. You can verify this by checking the metrics endpoint.</p> 
<p><strong>Validating the scraped data</strong></p> 
<p>You can validate that the metrics are being scraped by checking the metrics endpoint. First, ensure that you have port-forwarded a CSI node pod (where <code>&lt;ebs-csi-node-4cm75&gt;</code> is the name of our CSI node pod, but your naming will be different):</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">kubectl port-forward &lt;ebs-csi-node-4cm75&gt; 3302:3302 -n kube-system</code></pre> 
</div> 
<p>Then curl the endpoint:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">curl 127.0.0.1:3302/metrics</code></pre> 
</div> 
<p>This will yield output similar to the following sample:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash"># HELP nvme_collector_scrapes_total Total number of NVMe collector scrapes
# TYPE nvme_collector_scrapes_total counter nvme_collector_scrapes_total{instance_id="i-XXXXXXXXXXXXXXXXX"} 2
...</code></pre> 
</div> 
<p>You can also use <a href="https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-CW-usage-metrics.html">CloudWatch metrics to monitor your workspace</a> to validate that new data is being scraped.</p> 
<p>To visualize the scraped data, <a href="https://docs.aws.amazon.com/grafana/latest/userguide/Amazon-Managed-Grafana-setting-up.html">set up an Amazon Managed Grafana</a> workspace. Add the <a href="https://docs.aws.amazon.com/grafana/latest/userguide/AMP-adding-AWS-config.html">Prometheus workspace created previously as a data source</a> in Amazon Managed Grafana. See figure 2.</p> 
<p><img alt="The AWS Data Sources tab of the Amazon Managed Grafana workspace displays a list of supported AWS data sources. At the bottom of the list is Amazon Managed Service for Prometheus." class="alignnone size-full wp-image-57962" height="1015" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/11/06/figure2.png" width="2062" />Figure 2: Selecting Amazon Managed Service for Prometheus as a data source in Amazon Managed Grafana</p> 
<p>To validate the EBS detailed performance statistics are being sent to the Prometheus workspace, login to the Grafana instance created previously to visualize the data. Create a dashboard and in the metrics browser search for the relevant volume metrics. See figure 3.</p> 
<p><img alt="A visualization within Amazon Managed Grafana which shows the metric nvme_read_bytes_total. The graph shows a flat line to the 7 second mark, then a jump, followed by another flat line to the end of the visualization." class="alignnone size-full wp-image-57964" height="1154" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/11/06/figure3.png" width="2026" />Figure 3: Visualizing the nvme_read_bytes_total metric</p> 
<p><strong>Next steps</strong></p> 
<p>Once the Amazon EBS CSI driver in your EKS cluster has been updated to the latest version, these metrics will become available in your cluster. The AWS managed collector built into Amazon EKS makes it straightforward to begin collecting these detailed performance statistics. While these metrics are available free of charge, your Prometheus workspace is billed on the number of metrics ingested and the amount of storage used.</p> 
<p>In this blog post, we demonstrated how to begin to take advantage of EBS detailed performance statistics within your EKS clusters. Using these new metrics, you can better understand the performance of your latency-sensitive EKS workloads. These metrics give you real-time visibility into your volume’s I/O performance and can inform you when performance is impacted by a volume or when throughput limits are being exceeded. To get started, upgrade the EBS CSI Driver in your EKS clusters so you can begin to take advantage of these new metrics today.</p>
