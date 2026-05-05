---
title: "Optimizing metrics ingestion with Amazon Managed Service for Prometheus"
url: "https://aws.amazon.com/blogs/mt/optimizing-metrics-ingestion-with-amazon-managed-service-for-prometheus/"
date: "Thu, 18 Sep 2025 19:51:34 +0000"
author: "Sasi Kiran Malladi"
feed_url: "https://aws.amazon.com/blogs/mt/tag/amazon-managed-service-for-prometheus/feed/"
---
<p>Managing metrics collection at scale in complex cloud environments presents significant challenges for organizations, particularly when it comes to controlling costs and maintaining operational efficiency. As the volume of metrics grows exponentially with the expansion of container deployments and other cloud-native workloads, customers often struggle to balance comprehensive monitoring with resource optimization. This can lead to increased operational complexity and potential overspending on monitoring infrastructure.</p> 
<p>This blog is a part in a blog series on governance controls with <a href="https://docs.aws.amazon.com/prometheus/latest/userguide/what-is-Amazon-Managed-Service-Prometheus.html" rel="noopener" target="_blank">Amazon Managed Service for Prometheus</a>. In the first blog, <a href="https://aws.amazon.com/blogs/mt/optimizing-queries-with-amazon-managed-prometheus/" rel="noopener" target="_blank">Optimizing Queries with Amazon Managed Service for Prometheus</a>, we explored features like Query Logging and Query Thresholds that help optimize PromQL query patterns and costs. In this blog, we’ll explore how to set up cross-account metrics ingestion and establish governance controls with Amazon Managed Service for Prometheus.</p> 
<h2><strong>Overview</strong></h2> 
<p>Building on the centralized observability architecture scenario introduced in our first blog post, Example Corp, a multinational company, is collecting all platform and application metrics in Prometheus format with a centralized view from multiple AWS accounts.</p> 
<p>Example Corp maintains a central Amazon Managed Service for Prometheus workspace in their monitoring account, with managed collectors automatically scraping metrics from applications running across different AWS accounts. As shown in Figure 1, their architecture leverages <a href="https://aws.amazon.com/iam/" rel="noopener" target="_blank">AWS Identity and Access Management (IAM) roles</a> to establish secure, permission-based connections between accounts, eliminating the need for manually managing Prometheus scrapers in each account.</p> 
<div class="wp-caption aligncenter" id="attachment_62707" style="width: 3675px;">
 <img alt="Figure 1- Cross Account scraping using AWS managed collectors" class="wp-image-62707 size-full" height="2755" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2025/09/18/AMPcrossaccount.drawio-3.png" width="3665" />
 <p class="wp-caption-text" id="caption-attachment-62707">Figure 1: Cross Account scraping using AWS managed collectors</p>
</div> 
<p>While this centralized architecture significantly reduces operational overhead, Example Corp faces a growing challenge with metric volume control. As different application teams push increasing amounts of data to the central workspace, the Example Corp needs to add controls to optimize metrics ingestion. To address this challenge, it’s crucial to first understand how to monitor the Amazon Managed Service for Prometheus workspace effectively.</p> 
<h2>Insights into ingestion</h2> 
<p>Example Corp needs to gain visibility into their ingestion. Also, they want to set up alarms for when their ingestion starts to encounter issues. Amazon Managed Service for Prometheus now allows you to view applied quota values and their utilization for your Amazon Managed Service for Prometheus workspaces using <a href="https://docs.aws.amazon.com/servicequotas/latest/userguide/gs-request-quota.html" rel="noopener" target="_blank">AWS Service Quotas</a> and <a href="https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-CW-usage-metrics.html" rel="noopener" target="_blank">Amazon CloudWatch</a>.</p> 
<p>AWS Service Quotas allows you to quickly understand your applied service quota values and request increases in a few clicks. With Amazon CloudWatch usage metrics, you can create alarms to be notified when your Amazon Managed Service for Prometheus workspaces approach applied limits and visualize usage in CloudWatch dashboards. These metrics are essential for understanding your Amazon Managed Service for Prometheus workspace’s behavior and optimizing its performance.</p> 
<p>Below are some key metrics available for monitoring Amazon Managed Service for Prometheus. For all the available metrics, check out the <a href="https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-CW-usage-metrics.html" rel="noopener" target="_blank">CloudWatch metrics</a>:</p> 
<ol> 
 <li><strong>IngestionRate</strong>: Measures the volume of samples ingested per second, helping you understand your data inflow patterns</li> 
 <li><strong>ActiveSeries</strong>: Shows the number of unique time series currently active in your workspace</li> 
 <li><strong>QueryMetricsTPS</strong>: Indicates the number of queries processed per second</li> 
 <li><strong>RuleEvaluations</strong>: Tracks the number of alert rule evaluations</li> 
 <li><strong>DiscardedSamples</strong>: Tracks samples that couldn’t be ingested due to various constraints</li> 
</ol> 
<p><img alt="Figure 2: Amazon Managed Service for Prometheus service quotas using AWS Service Quotas" class="size-full wp-image-62575" height="2271" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2025/09/05/Figure-2-Amazon-Managed-Service-for-Prometheus-service-quotas-using-AWS-Service-Quotas.png" width="5354" /><em style="font-size: 14px;">Figure 2: Amazon Managed Service for Prometheus service quotas using AWS Service Quotas</em></p> 
<p>Below is an example showing the applied quotas and utilization of Amazon Managed Service for Prometheus resource ‘Active series per workspace’:</p> 
<div class="wp-caption alignnone" id="attachment_62577" style="width: 3676px;">
 <img alt="Figure 3: Quotas for Amazon Managed Service for Prometheus resource ‘Active series per workspace’ using AWS Service Quotas" class="size-full wp-image-62577" height="1448" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2025/09/05/Figure-3-Quotas-for-Amazon-Managed-Service-for-Prometheus-resource-‘Active-series-per-workspace-using-AWS-Service-Quotas.png" width="3666" />
 <p class="wp-caption-text" id="caption-attachment-62577"><em>Figure 3: Quotas for Amazon Managed Service for Prometheus resource ‘Active series per workspace’ using AWS Service Quotas</em></p>
</div> 
<p>With <a href="https://aws.amazon.com/grafana/" rel="noopener" target="_blank">Amazon Managed Grafana</a>, Example Corp can add CloudWatch as the data source to query, correlate and visualize metrics, logs and traces in a dashboard.</p> 
<p><img alt="Figure 4 - Amazon Managed Grafana dashboard to visualize Amazon Managed Service for Prometheus workspace resources utilization.png" class="size-full wp-image-62578" height="504" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2025/09/05/Figure-4-Amazon-Managed-Grafana-dashboard-to-visualize-Amazon-Managed-Service-for-Prometheus-workspace-resources-utilization.png" width="1538" /><em style="font-size: 14px;">Figure 4: Amazon Managed Grafana dashboard to visualize Amazon Managed Service for Prometheus workspace resources utilization</em></p> 
<div class="wp-caption alignnone" id="attachment_62589" style="width: 1608px;">
 <img alt="Figure 5: Insights into Amazon Managed Service for Prometheus ingestion using Amazon Managed Grafana dashboard" class="size-full wp-image-62589" height="394" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2025/09/05/Figure-5-Insights-into-Amazon-Managed-Service-for-Prometheus-ingestion-using-Amazon-Managed-Grafana-dashboard-1.png" width="1598" />
 <p class="wp-caption-text" id="caption-attachment-62589">Figure 5: Insights into Amazon Managed Service for Prometheus ingestion using Amazon Managed Grafana dashboard</p>
</div> 
<p>The <strong>DiscardedSamples</strong> metric is particularly crucial as it helps identify potential issues with your metric collection. Samples might be discarded for several reasons:</p> 
<ol> 
 <li>Exceeding active series limits when applications generate more unique time series than your workspace can handle</li> 
 <li>Reaching ingestion rate limits during traffic spikes or when multiple systems send large volumes of metrics simultaneously</li> 
 <li>Receiving out-of-order samples or samples with timestamps outside the acceptable time window, often due to clock synchronization issues</li> 
</ol> 
<p>By monitoring these metrics, Example Corp can effectively track performance, plan capacity, and manage costs across their Amazon Managed Service for Prometheus workspaces. This visibility becomes particularly important as the observability team needs to control the number of active series per label to optimize costs and maintain workspace performance. While monitoring provides insights into workspace usage, Example Corp needs more granular control over metric ingestion to ensure efficient resource utilization.</p> 
<h2>Label-based active series limits</h2> 
<p>Amazon Managed Service for Prometheus <a href="https://aws.amazon.com/about-aws/whats-new/2025/04/amazon-managed-service-prometheus-label-based-series-limits/" rel="noopener" target="_blank">now supports label-based active series limits</a> within workspaces. This feature enables Example Corp to manage active series volume by setting specific limits for different metric producers, whether they are applications, services, or teams sharing a workspace.</p> 
<p>Using custom quotas based on labels, Example Corp can protect critical metrics from unexpected surges in workloads. This is particularly valuable in multi-tenant environments or when isolating metrics by application tier, as it helps maintain predictable resource consumption and cost-effectiveness.</p> 
<p>The feature works by defining rules using key-value pairs. For example, Example Corp can set specific quotas for metrics with labels like <strong>{app=”payment-service”, environment=”prod”}</strong>. If a production payment service experiences a sudden metric spike due to a configuration error, only that specific label set’s metrics will be throttled, leaving other workloads unaffected.</p> 
<p>This targeted approach delivers three key benefits:</p> 
<ol> 
 <li>Minimizes operational risk by preventing noisy neighbors from overwhelming the workspace</li> 
 <li>Enables teams to enforce fair usage policies without manual intervention</li> 
 <li>Maintains high availability for mission-critical systems while optimizing costs</li> 
</ol> 
<h2>Configure label sets and limits</h2> 
<p>The Example Corp team can configure label sets limits with few simple steps using the AWS console:</p> 
<ol> 
 <li>Open the <a href="https://console.aws.amazon.com/prometheus/" rel="noopener" target="_blank">Amazon Managed Service for Prometheus console</a>.</li> 
 <li>In the upper left corner of the page, choose the menu icon and then choose <strong>All workspaces</strong>.</li> 
 <li>Choose the <strong>Workspace ID</strong> of the workspace.</li> 
 <li>Choose the <strong>Workspace configurations</strong> tab.</li> 
 <li>To add or modify label sets and their active series limits, choose <strong>Edit</strong> in the <strong>Label sets</strong> section. Then do the following: 
  <ul> 
   <li>(Optional) Enter a value in <strong>Default bucket limit</strong> to set a limit on the maximum number of active time series that can be ingested in the workspace, counting only time series that don’t match any defined label set.</li> 
   <li>To define a label set, enter an active time series limit for the new label set under <strong>Active series limit</strong>. Then, enter a label and value for one label that will be used in the label set, and choose <strong>Add label</strong>.</li> 
   <li>(Optional) To define another label set, choose <strong>Add another label set</strong> and repeat the previous steps.</li> 
  </ul> </li> 
</ol> 
<div class="wp-caption alignnone" id="attachment_62590" style="width: 3585px;">
 <img alt="Figure 6: Modify Amazon Managed Service for Prometheus Workspace configurations" class="size-full wp-image-62590" height="1567" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2025/09/05/Figure-6-Modify-Amazon-Managed-Service-for-Prometheus-Workspace-configurations.png" width="3575" />
 <p class="wp-caption-text" id="caption-attachment-62590"><em>Figure 6: Modify Amazon Managed Service for Prometheus Workspace configurations</em></p>
</div> 
<p>Alternatively, you can use the <a href="https://docs.aws.amazon.com/cli/latest/reference/amp/update-workspace-configuration.html" rel="noopener" target="_blank">AWS CLI</a> to configure the label sets limits and retention period as below:</p> 
<p>Create a JSON file as below and replace with your appropriate values.</p> 
<pre><code class="lang-json">{
    "workspaceId": "&lt;WORKSPACE_ID&gt;",
    "limitsPerLabelSet": [
        {
            "limits": {
                "maxSeries":&lt;ENTER_VALUE&gt;
            },
            "labelSet": {
                "&lt;KEY_NAME1&gt;": "&lt;KEY_VALUE1&gt;",
                "&lt;KEY_NAME2&gt;": "&lt;KEY_VALUE2&gt;"
            }
        }
    ],
    "retentionPeriodInDays":&lt;ENTER_VALUE&gt;
}</code></pre> 
<p><code>aws amp update-workspace-configuration --workspace-id &lt;WORKSPACE_ID&gt; --cli-input-json file://"ENTER_NAME".json</code></p> 
<p><em>Tip: Label sets let you group metrics by specific dimensions (like application, region or service), so choose labels that align with your organizational needs.</em></p> 
<p>Once you’ve added all desired label sets and optional defaults, save your changes. Your workspace will now enforce these limits, helping you maintain better control over data ingestion.</p> 
<div class="wp-caption alignnone" id="attachment_62592" style="width: 4517px;">
 <img alt="Figure 7: Configure label sets in Amazon Managed Service for Prometheus" class="size-full wp-image-62592" height="2750" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2025/09/05/Figure-7-Configure-label-sets-in-Amazon-Managed-Service-for-Prometheus-1.png" width="4507" />
 <p class="wp-caption-text" id="caption-attachment-62592"><em>Figure 7: Configure label sets in Amazon Managed Service for Prometheus</em></p>
</div> 
<p>Example Corp should also update the Scraper configuration to match the newly created/updated Label Sets. Follow below steps to update <a href="https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-collector.html" rel="noopener" target="_blank">AWS Managed Collectors</a>:</p> 
<ol> 
 <li>In the left navigation, choose <strong>All Managed Collector</strong></li> 
 <li>Choose the <strong>ID</strong> of the <strong>Scraper</strong></li> 
 <li>Select <strong>Update</strong></li> 
 <li>Add the <strong>external_labels</strong> matching the above the <strong>Label Sets</strong> as per your needs</li> 
 <li>When finished editing, select <strong>Update</strong></li> 
</ol> 
<div class="wp-caption alignnone" id="attachment_62593" style="width: 3511px;">
 <img alt="Figure 8: Update Scraper configuration in Amazon Managed Service for Prometheus" class="size-full wp-image-62593" height="1732" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2025/09/05/Figure-8-Update-Scraper-configuration-in-Amazon-Managed-Service-for-Prometheus.png" width="3501" />
 <p class="wp-caption-text" id="caption-attachment-62593"><em>Figure 8: Update Scraper configuration in Amazon Managed Service for Prometheus</em></p>
</div> 
<p>Alternatively, you can use the AWS CLI to update the scraper as below:</p> 
<ol> 
 <li>Update the <a href="https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-collector-how-to.html#AMP-collector-configuration" rel="noopener" target="_blank">sample Scraper configuration</a> with external labels matching the above label sets.</li> 
 <li>Convert the yaml file to base64 format using any converters. As an example: <code>base64 -w 0 &lt;SCRAPER&gt;.yml</code></li> 
 <li>Copy the output from that command and paste it into the below command:</li> 
</ol> 
<p><code>aws amp update-scraper --scraper-id &lt;ENTER_VALUE&gt; --scrape-configuration configurationBlob=&lt;PASTE_SCRAPER_BLOB&gt;</code></p> 
<h2></h2> 
<h2>Understanding the Amazon CloudWatch Metrics for label sets limits</h2> 
<p>As <a href="https://aws.amazon.com/about-aws/whats-new/2025/04/amazon-managed-service-prometheus-label-based-series-limits/" rel="noopener" target="_blank">announced</a>&nbsp;earlier, there are four <a href="https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-CW-usage-metrics.html" rel="noopener" target="_blank">CloudWatch metrics</a> under the AWS/Prometheus namespace to enhance visibility into your Amazon Managed Service for Prometheus workspace’s performance and resource usage. These metrics include:</p> 
<ol> 
 <li><strong>ActiveSeriesPerLabelSet</strong>: Tracks the current number of active time series for each user-defined label set, helping you monitor real-time usage against configured limits.</li> 
 <li><strong>ActiveSeriesLimitPerLabelSet</strong>: Shows the configured limit for active series per label set, enabling quick comparisons between actual usage and allowed thresholds.</li> 
 <li><strong>IngestionRatePerLabelSet</strong>: Measures the data ingestion rate (samples/sec) for each label set, useful for identifying high-volume metrics or potential bottlenecks.</li> 
 <li><strong>DiscardedSamplesPerLabelSet</strong>: Reveals how many samples were dropped due to exceeded limits, now with granular reasons: hitting a label set limit (<strong>per_labelset_series_limit</strong>), per-metric limit (<strong>per_metric_series_limit</strong>), or global user limit (<strong>per_user_series_limit</strong>). 
  <ul> 
   <li>It’s important to note that these metrics will only be available if they have populated data points.</li> 
  </ul> </li> 
</ol> 
<p>These metrics empower teams to proactively manage scalability, troubleshoot ingestion issues, and enforce cost controls. For example, spikes in <strong>DiscardedSamplesPerLabelSet</strong> with the <strong>per_labelset_series_limit</strong> reason signal a need to adjust label set limits, while tracking IngestionRatePerLabelSet helps optimize high-cardinality metrics. By correlating these metrics, Example Corp can gain actionable insights to balance performance, cost, and data retention, while ensuring critical metrics are prioritized, and infrastructure stays efficient.</p> 
<h2></h2> 
<h2>Sample Dashboard</h2> 
<div class="wp-caption alignnone" id="attachment_62597" style="width: 1930px;">
 <img alt="Figure 9: Visualize Amazon Managed Service for Prometheus Label Set based active series limits using Amazon Managed Grafana dashboard" class="size-full wp-image-62597" height="958" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2025/09/05/Figure-9-Visualize-Amazon-Managed-Service-for-Prometheus-Label-Set-based-active-series-limits-using-Amazon-Managed-Grafana-dashboard-2.png" width="1920" />
 <p class="wp-caption-text" id="caption-attachment-62597"><em>Figure 9: Visualize Amazon Managed Service for Prometheus Label Set based active series limits using Amazon Managed Grafana dashboard</em></p>
</div> 
<p>You can now download this sample dashboard template from the <a href="https://github.com/aws-observability/aws-observability-accelerator/blob/main/artifacts/grafana-dashboards/amp/amp-workspace-dashboard.json" rel="noopener" target="_blank">AMP Workspace Dashboard</a> and import it as a dashboard into your Amazon Managed Grafana. This dashboard also has the capabilities to visualize multi-account and multiple Amazon Managed Service for Prometheus workspaces use cases.</p> 
<p><strong>Sample queries used to build this dashboard:</strong></p> 
<p><strong>Query 1</strong> computes the average of the ActiveSeriesLimitPerLabelSet metric from the AWS/Prometheus namespace. It uses the schema () function to access metric data for the LabelSet and Workspace dimensions. The group by LabelSet clause segments the results to identify which LabelSets are consuming the most active series resources, helping in high-cardinality analysis.</p> 
<p><code>SELECT AVG(ActiveSeriesLimitPerLabelSet) FROM SCHEMA("AWS/Prometheus", LabelSet,Workspace) GROUP BY LabelSet</code></p> 
<p>Optionally, you could also use the <a href="https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/using-search-expressions.html" rel="noopener" target="_blank">Search expression</a> as in the below query to get insights for all the label sets and their metrics across all the workspaces.</p> 
<p><code>SEARCH('{AWS/Prometheus,LabelSet,Workspace}', 'Sum', 300)</code></p> 
<p><strong>Query 2</strong> calculates the average number of DiscardSamplesPerLabelSet from the AWS/Prometheus namespace. It uses the schema () function to analyze data across the LabelSet, Reason, and Workspace dimensions. By grouping results by LabelSet, it identifies which label combinations are most frequently causing sample drops, helping diagnose issues like label churn or cardinality limits.</p> 
<p><code>SELECT AVG(DiscardedSamplesPerLabelSet) FROM SCHEMA("AWS/Prometheus", LabelSet,Reason,Workspace) GROUP BY LabelSet</code></p> 
<p><strong>Query 3</strong> computes the total of DiscardedSamples from the AWS/Prometheus namespace. It groups the results by the Reason dimension to categorize why samples were discarded. This helps identify root causes such as high cardinality, remote write rejections, or ingestion throttling affecting Prometheus data reliability.</p> 
<p><code>SELECT SUM(DiscardedSamples) FROM "AWS/Prometheus" GROUP BY Reason</code></p> 
<p><strong>Query 4</strong> calculates the average of ActiveSeriesPerLabelSet from the AWS/Prometheus namespace, scope to the LabelSet and Workspace dimensions. It filters the data to only include timeseries with the specific LabelSet. Replace LabelSet name as per your needs. This helps monitor the time series activity for a specific application, enabling targeted analysis of its metric cardinality and usage.</p> 
<p><code>SELECT AVG(ActiveSeriesPerLabelSet) FROM SCHEMA("AWS/Prometheus", LabelSet,Workspace) WHERE LabelSet = '{app="payment-service", environment="prod"}'</code></p> 
<p><strong>Query 5</strong> calculates the average of IngestionRatePerLabelSet. This helps assess the metric ingestion rate for a specific application to identify high-volume metric sources and potential ingestion bottlenecks.</p> 
<p><code>SELECT AVG(IngestionRatePerLabelSet) FROM SCHEMA("AWS/Prometheus", LabelSet,Workspace) WHERE LabelSet = '{app="payment-service", environment="prod"}'</code></p> 
<p><strong>Query 6</strong> calculates the total of ResourceCount from the AWS/Usage namespace for the ‘Prometheus’ service. It groups the results by the Resource dimension to break down usage across different Prometheus-related resource types. This helps monitor the overall Prometheus resource consumption for cost and capacity management.</p> 
<p><code>SELECT SUM(ResourceCount) FROM "AWS/Usage" WHERE Service = 'Prometheus' GROUP BY Resource</code></p> 
<h2></h2> 
<h2>Conclusion</h2> 
<p>In this blog, we explored how effectively managing metric ingestion at scale is critical for organizations like Example Corp, where uncontrolled data growth can lead to operational inefficiencies and escalating costs. We showed how Amazon Managed Service for Prometheus addresses these challenges with its new label-based active series limits, empowering teams to enforce granular controls over metric ingestion. By setting custom quotas tied to specific labels, organizations can proactively mitigate risks like noisy neighbor disruptions, ensure equitable resource allocation, and safeguard mission-critical systems from unexpected metric surges. We also demonstrated how these features not only enhance cost predictability by preventing resource overconsumption but also streamlines observability management in multi-tenant or tiered application architectures. With label-based throttling, teams maintain flexibility to isolate and prioritize workloads while sustaining high availability and operational resilience.</p> 
<p>Check out the latest <a href="https://aws.amazon.com/about-aws/whats-new/2025/09/amazon-managed-service-prometheus-quota-visibility-aws-service-quotas-cloudwatch/" rel="noopener" target="_blank">Amazon Managed Service for Prometheus announcement</a>, <a href="https://docs.aws.amazon.com/prometheus/latest/userguide/AMP_quotas.html" rel="noopener" target="_blank">Amazon Managed Service for Prometheus service quotas</a> and <a href="https://aws.amazon.com/about-aws/whats-new/2025/04/amazon-managed-service-prometheus-label-based-series-limits/" rel="noopener" target="_blank">Amazon Managed Service for Prometheus label-based active series limits</a>. These updates give you a comprehensive view of health, performance, and quota utilization across your workspaces.</p> 
<p>To learn more about AWS Observability, checkout the <a href="https://aws-observability.github.io/observability-best-practices/" rel="noopener" target="_blank">AWS Observability Best Practices Guide</a>.</p> 
<p>To get hands-on experience with AWS Observability services, check out the <a href="https://catalog.workshops.aws/observability/en-US" rel="noopener" target="_blank">One Observability Workshop</a>.</p>
