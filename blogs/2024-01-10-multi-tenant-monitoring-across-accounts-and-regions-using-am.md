---
title: "Multi-tenant monitoring across accounts and regions using Amazon Managed Service for Prometheus"
url: "https://aws.amazon.com/blogs/mt/multi-tenant-monitoring-across-accounts-and-regions-using-amazon-managed-for-prometheus/"
date: "Wed, 10 Jan 2024 20:23:21 +0000"
author: "Manjit Chakraborty"
feed_url: "https://aws.amazon.com/blogs/mt/tag/amazon-managed-service-for-prometheus/feed/"
---
<p><em>In this guest blog post, Nauman Noor (Managing Director), Fabio Dias (Cloud Developer), and Dylan Alibay (Cloud Developer) from the platform engineering team at State Street discuss their use of Amazon Managed Prometheus and AWS Distro for OpenTelemetry to enable monitoring in a multi-tenant, multi-account, and multi-region environment.</em></p> 
<p>In the ever-evolving financial services landscape, <a href="http://statestreet.com/">State Street Corporation</a> is a world leader in investment services, investment management, and investment research and trading for institutional investors.</p> 
<p>State Street operates in a complex AWS environment that includes multiple tenants, regions, and accounts, which makes observability a challenge.</p> 
<p>In this blog post, we will cover how State Street used AWS Observability Services to aggregate their observability data into a single pane of glass by leveraging <a href="https://aws.amazon.com/otel/">AWS Distro for OpenTelemetry (ADOT)</a> and <a href="https://aws.amazon.com/prometheus/">Amazon Managed Service for Prometheus</a>.</p> 
<h3><strong>Solution overview</strong></h3> 
<p>When evaluating different observability solutions, State Street wanted a service that would have low overhead to manage and wouldn’t need a large initial investment. They were looking for an option that minimized management burden and upfront costs.</p> 
<p>The high-level architecture for the solution is illustrated in Figure 1.</p> 
<p>State Street used A<a href="https://aws.amazon.com/ecs/">mazon Elastic Container Service (ECS)</a> &nbsp;to run processing tasks in a scalable way using AWS Distro for OpenTelemetry (ADOT). Amazon Managed Prometheus (AMP) provided data persistence through Grafana dashboards, enabling data visualization and analysis. The key components were ECS for task processing, ADOT for scalability, AMP for data storage, and Grafana for visualization.</p> 
<p><img alt="- High level end-to-end architecture of the solution" class="aligncenter wp-image-48263" height="1693" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/01/04/Picture1-3.png" width="2500" /></p> 
<p style="text-align: center;"><em>Figure 1 – High level end-to-end architecture of the solution</em></p> 
<p>State Street collected performance metrics from the resources across their monitored environments, including their on-premises data centers and multiple AWS accounts spanning several regions. They aggregated this monitoring data by funneling it into a Central Account. Centralizing the data in this way improved reliability and security while also simplifying analysis across the different environments being monitored.</p> 
<p>The details this workflow is divided in the following sections: Metric Collection, Aggregation, and Presentation.</p> 
<p><span style="text-decoration: underline;"><strong>Metric Collection</strong></span><br /> State Street leveraged a combination of <a href="https://aws.amazon.com/cloudops/monitoring-and-observability/?whats-new-cards.sort-by=item.additionalFields.postDateTime&amp;whats-new-cards.sort-order=desc&amp;blog-posts-cards.sort-by=item.additionalFields.createdDate&amp;blog-posts-cards.sort-order=desc">AWS native metrics solutions</a> and code instrumentation to generate the metrics. The scenario can be split into three main categories:</p> 
<p>1. Native AWS service metrics:</p> 
<p>– Collected from Amazon CloudWatch using <a href="https://github.com/nerdswords/yet-another-cloudwatch-exporter">YACE (Yet-Another-CloudWatch-Exporter)</a></p> 
<p>– YACE exposes an ADOT compatible endpoint that queries and caches metrics from AWS API</p> 
<p>2. Amazon EC2 and Amazon ECS metrics:</p> 
<p>– Uses open-source exporters like <a href="https://github.com/prometheus/node_exporter">node_exporter</a> and <a href="https://github.com/google/cadvisor">cAdvisor</a></p> 
<p>– Provides higher frequency and additional metrics</p> 
<p>3. Ad-hoc metrics:</p> 
<p>– Includes language specific metrics like JMX, python via <a href="https://opentelemetry.io/docs/instrumentation/python/automatic/">opentelemetry-exporter-otlp</a></p> 
<p>– Also includes business-oriented metrics defined at the application level</p> 
<p>This information needs to be collected and processed, which is done by ECS services running ADOT configured as scrapers. Each scraper periodically queries a subset of existing resources, enriching them with identification metadata, and forward those metrics to the Central Account.</p> 
<p>The architecture diagram showcasing this phase is illustrated in Figure 2.</p> 
<p><img alt="Diagram of scrapers in a monitored account" class="aligncenter wp-image-48264 size-full" height="845" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/01/04/Picture2-3.png" width="981" /></p> 
<p style="text-align: center;"><em>Figure 2 – Diagram of scrapers in a monitored account</em></p> 
<p>To optimize this process, the team deployed the scrapers in the same availability zones as the resources they monitor. This reduced latency and data transfer costs. For large environments, they split the scope into multiple scrapers, each responsible for querying specific resources within the configured measurement interval. They are configured for <a href="https://docs.aws.amazon.com/prometheus/latest/userguide/Send-high-availability-data-ADOT.html">high availability</a> with each Amazon ECS service configured to run multiple copies of each scraper. This ensures observability during maintenance activities and downtime. They also leveraged the <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-circuit-breaker.html">deployment circuit breaker</a> features of Amazon ECS to provide stability during service updates.</p> 
<p><span style="text-decoration: underline;"><strong>Aggregation</strong></span><br /> The Central Account receives metrics from several environments via a set of dedicated ECS services running ADOT. Those services are configured to receive telemetry via HTTPS and persist it on the AMP Workspaces. State Street calls those services middleware, and they enable a seamless integration with on-premises data sources, since they support open industry standards.</p> 
<p>This approach has advantages for the AWS environments as well:</p> 
<p>1. Low Authentication Overhead: Requires the set-up of basic authentication to ensure metrics are ingested only from authorized sources</p> 
<p>2. Scalable: Reduced ingestion pressure on AMP Workspace by batching requests in the middleware</p> 
<p>3. Flexible: Provides an abstraction layer between the resources and analysis, allowing for flexibility in choice of solutions for Metric Collection and Presentation</p> 
<p>The architecture diagram for this phase is illustrated in Figure 3.</p> 
<p><img alt="Diagram of the middleware on the Central Account" class="size-full wp-image-48265 aligncenter" height="541" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/01/04/Picture3-2.png" width="979" /></p> 
<p style="text-align: center;"><em>Figure 3 – Diagram of the middleware on the Central Account</em></p> 
<p><strong>Handling dynamic loads</strong><br /> Metric volume can fluctuate significantly due to new deployments, failovers, etc. The middleware needs to be responsive to avoid dropping data points during crucial observability events.</p> 
<p>To ensure high availability, State Street leveraged <a href="https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-auto-scaling.html">ECS Service Auto Scaling</a>.</p> 
<p>However, the traditional memory-based scaling is inadequate as during regular operation, data flows out as quickly as it flows in without the need to buffer/cache the data. An increase in memory usage would indicate backpressure, eventually leading to memory starvation and the task being replaced.</p> 
<p>State Street adopted auto-scaling based on request count per load balancer target instead. They ran load tests to establish how many requests per second they could reliably handle, as this can vary depending on the resources and configuration. An example of the autoscaling in action is shown in Figure 4, where the number of requests over time is superimposed with the scaling events. The dotted line represents the configured limit of requests per second.</p> 
<p><img alt="Autoscaling in Action, ALB Request Count per Middleware Task" class="wp-image-48266 aligncenter" height="1070" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/01/04/Picture4-3.png" width="2000" /></p> 
<p style="text-align: center;"><em>Figure 4 – Autoscaling in Action, ALB Request Count per Middleware Task</em></p> 
<p>The highlighted points correspond to:</p> 
<p>A. This is regular operations under the threshold with only 1 task: Middleware 1.</p> 
<p>B. There is an increase in requests, going over the configured threshold.</p> 
<p>C. A new task (Middleware 2) is added, which reduces the load per task into manageable levels.</p> 
<p>D. Once the total number of requests decreases the additional task is no longer required and scales-in.</p> 
<p>To further improve the availability of the solution through extreme surge in requests, the team enabled the <a href="https://github.com/open-telemetry/opentelemetry-collector/tree/main/processor/memorylimiterprocessor#Suggestion%20:%20To%20further%20improve%20the%20availability%20of%20the%20solution%20through%20extreme%20surge%20in%20requests,%20we%20enabled%20the%20memory%20limiter%20processor%20available%20in%20ADOT.%20The%20memory%20limiter%20monitors%20the%20memory%20usage%20and%20pessimistically%20drops%20data%20points%20so%20as%20to%20not%20overwhelm%20the%20running%20task%20when%20its%20memory%20usage%20is%20%20close%20to%20the%20maximum.%20This%20should%20rarely%20occur%20as%20we%20also%20configured%20a%20safety%20margin%20on%20the%20scale-out%20threshold%20to%20cover%20for%20the%20start%20up%20time%20of%20new%20middleware%20tasks.%0Dmemory-limiter-processor">memory limiter processor</a> available in ADOT. The memory limiter monitors the memory usage and pessimistically drops data points so as to not overwhelm the running task when its memory usage is close to the maximum. This should rarely occur as they also configured a safety margin on the scale-out threshold to cover for the start up time of new middleware tasks.</p> 
<p><span style="text-decoration: underline;"><strong>Presentation</strong></span><br /> The middleware then persists the data into Amazon Managed Service for Prometheus Workspaces, a logical space dedicated to the storage and querying of Prometheus metrics. State Street used the <a href="https://github.com/open-telemetry/opentelemetry-collector/tree/main/processor/memorylimiterprocessor#Suggestion%20:%20To%20further%20improve%20the%20availability%20of%20the%20solution%20through%20extreme%20surge%20in%20requests,%20we%20enabled%20the%20memory%20limiter%20processor%20available%20in%20ADOT.%20The%20memory%20limiter%20monitors%20the%20memory%20usage%20and%20pessimistically%20drops%20data%20points%20so%20as%20to%20not%20overwhelm%20the%20running%20task%20when%20its%20memory%20usage%20is%20%20close%20to%20the%20maximum.%20This%20should%20rarely%20occur%20as%20we%20also%20configured%20a%20safety%20margin%20on%20the%20scale-out%20threshold%20to%20cover%20for%20the%20start%20up%20time%20of%20new%20middleware%20tasks.%0Dmemory-limiter-processor">Prometheus Remote Write Exporter</a>. Metrics stored into the workspace are then used for dashboards and alerting using Grafana and native AMP alerting.</p> 
<p>Figure 5 is a snapshot of this visualization, showing in green the ingestion rate of the AMP and in yellow the number of middleware ECS tasks.</p> 
<p><img alt="Metrics visualization using Grafana" class="aligncenter wp-image-48267" height="737" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/01/04/Picture5-2.png" width="1500" /></p> 
<p style="text-align: center;"><em>Figure 5 – Metrics visualization using Grafana</em></p> 
<p><strong>Tenant isolation</strong><br /> As a multi-tenant environment, one of the requirements was to control access to tenant information, including metrics.</p> 
<p>To provide extra degrees of isolation, State Street replicated the entire solution for different groupings of tenants, segregating the metrics at the start of the process. They dedicated scrapers, middleware, and workspaces to each tenant group. When scraping, they filter the metrics to only include the applicable resources for each tenant group. Then, they route the metrics for each tenant group separately. Furthermore, they only grant each tenant group access to their own dedicated workspace – no group can access metrics for the other groups.</p> 
<h3><strong>Conclusion</strong></h3> 
<p>Even in simple settings, achieving comprehensive observability can be a challenge. State Street faced this challenge when tasked with consolidating their AWS cloud, on-premises environments, and multiple multi-tenant AWS accounts across regions into one cohesive and robust system.</p> 
<p>To overcome this, they leveraged AWS Distro for OpenTelemetry, Amazon Managed Service for Prometheus, and Amazon Managed Grafana to build a monitoring framework that enables easy and effective resource monitoring. By utilizing these purpose-built AWS tools for OpenTelemetry and Prometheus, they were able to create a unified observability solution that spans environments, accounts, and regions. This allows their users to monitor resources across their complex, multi-faceted landscape.</p> 
<p>To learn more about AWS Observability services, please check the below resources:</p> 
<p>– <a href="https://catalog.workshops.aws/observability/en-US">One Observability Workshop</a></p> 
<p>– <a href="https://aws-observability.github.io/observability-best-practices/">AWS Observability Best Practices Guide</a></p> 
<p>–<a href="https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-onboard-ingest-metrics-OpenTelemetry-ECS.html"> Set up metrics ingestion from Amazon ECS using AWS Distro for OpenTelemetry</a></p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Nauman Noor" class="size-full wp-image-48377 aligncenter" height="300" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/01/08/Nauman-300x300.jpg" width="300" />
  </div> 
  <h3 class="lb-h4">Nauman Noor</h3> 
  <p>Nauman Noor leads the global platform engineering team enabling AWS for State Street and its clients. When not working, he spends time with his family exploring New York.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Fabio Dias" class="size-full wp-image-48377 aligncenter" height="300" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/01/04/profile-300x200.jpg" width="300" />
  </div> 
  <h3 class="lb-h4">Fabio Dias</h3> 
  <p>Fabio Dias is a Cloud Developer at State Street based out of Toronto. He is part of the team that maintains public cloud infrastructure services. Avid reader and amateur photographer, whenever possible.</p> 
 </div> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="Dylan Alibay" class="size-full wp-image-48377 aligncenter" height="300" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/01/08/IMG_0738-300x300.jpg" width="300" />
  </div> 
  <h3 class="lb-h4">Dylan Alibay</h3> 
  <p>Dylan Alibay is a Cloud Developer at State Street in Toronto, focusing on AWS cloud observability within the platform engineering team. He brings practical experience to maintain and improve the company’s cloud infrastructure. Outside of work, he finds joy in playing guitar, experimenting with baking recipes and spending quality time with his partner.</p> 
 </div> 
</footer>
