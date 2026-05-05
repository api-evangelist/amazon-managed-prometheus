---
title: "VTEX scales to 150 million metrics using Amazon Managed Service for Prometheus"
url: "https://aws.amazon.com/blogs/mt/scaling-to-150-million-metrics-using-amazon-managed-service-for-prometheus/"
date: "Sun, 10 Mar 2024 22:43:00 +0000"
author: "Jose Augusto Ferronato"
feed_url: "https://aws.amazon.com/blogs/mt/tag/amazon-managed-service-for-prometheus/feed/"
---
<p>VTEX is a multi-tenant platform with a distributed engineering operation. Observing hundreds of services in real time in an efficient manner is a technical challenge for the business. In this blog, we will show how VTEX created a resilient open source-based architecture aligned with a sharding strategy, using Amazon Managed Service for Prometheus (AMP) to ingest and store metrics. This solution increased the visibility of engineering teams, reduced the average incident resolution time, and made the investment in cloud infrastructure needed to implement observability within VTEX more efficient, reaching ~41% of reduction on investments in observability.</p> 
<h1>Introduction</h1> 
<p>VTEX started its operation in 2000 as a B2B software focused on the textile sector. Since then, there have been several changes that have influenced the company’s culture. Starting in 2007, VTEX entered the e-commerce market, resulting in the VTEX product that we know today. Along the way, there were several evolutions to achieve an increasingly efficient, resilient, and innovative product for the customer. As VTEX grew, it moved from monolithic to microservices, as it migrated its computing model from data centers to the cloud, boosting efficiency and scalability. Today, VTEX is a global company with more than 3,400 active customers over 38 countries that trust VTEX to accelerate and transform their retail businesses.</p> 
<h1>Problem</h1> 
<p>Since VTEX is a global company with hundreds of services, it has invested in monitoring and observability to detect application errors, bottlenecks, and anomalous behaviors that may indicate fraud, among other things. For many years, the company used a single software vendor to provide observability. It has served its purpose over the years, however, due to the market momentum experienced by VTEX and its constant engineering innovation, this centralized observability model no longer made sense for the company.</p> 
<p>When this centralized tool experienced failures due to unavailability or delayed telemetry ingestion, the VTEX engineering team was consequently left without visibility of its metrics, logs, and application traces. This had a direct impact on the efficiency of VTEX’s incident recovery time. In addition, these telemetry signals (logs, metrics, and tracks), individually, require different configurations. A centralized tool cannot be the best tool for all telemetry signals. Therefore, it’s not logically efficient to rely on a single tool for all of these types of data.</p> 
<p>In addition, the lack of control made the cost of this tool unpredictable. Imagine that any major sales event, such as Black Friday, in any country can make the VTEX platform scale. As the systems scale, the volume of telemetry increases linearly with the number of resources being monitored. The consequence was a cost disproportionate to the investment made in the tool. In this blog, we will discuss how VTEX looked at available solutions to help it scale its metrics pipeline.</p> 
<h1>Options</h1> 
<p>To solve the issues mentioned in the previous section, VTEX considered three community solutions and one managed by AWS: Cortex, VictoriaMetrics, TimescaleDB, and Amazon Managed Service for Prometheus.</p> 
<p>Cortex continued to be an attractive option due to its maturity level as an open-source CNCF project and its features for using Prometheus at scale, such as remote write/read, PromQL, recording rules and Alertmanager support. In addition, its integration with Prometheus would allow VTEX to easily monitor its systems and receive alerts in case of problems. There is another factor in the balance, which is to already have an infrastructure running with Cortex internally. At the end of the day, VTEX could assess whether the engineering investment could mitigate the problems found and avoid going to a new tool.</p> 
<p>VictoriaMetrics is a popular alternative to Cortex, with advantages that include high scalability, support for different data formats, high performance, high availability support, and low resource consumption. However, there are some drawbacks to consider, such as limited compatibility with other solutions and lower adoption. It uses its own custom database engine. It is a high-performance, column-oriented time series database designed to handle large volumes of data with high ingestion rates and query performance. The database engine is optimized for efficient compression and provides a number of advanced features, such as fast indexing, data retention policies, data replication, and high availability.</p> 
<p>TimescaleDB is a relational time series database integrated with PostgreSQL. It is scalable, resilient, and can support complex SQL queries. TimescaleDB is well suited for high-volume data environments where real-time and historical data analysis is required. One of the main advantages of TimescaleDB is that it allows users to use standard SQL to perform queries on their time series data.</p> 
<h1>Why Amazon Managed Service for Prometheus?</h1> 
<p>Amazon Managed Service for Prometheus is an efficient solution for monitoring and alerting systems, applications, and services. Here are the reasons VTEX chose AMP for its monitoring solution:</p> 
<ul> 
 <li><strong>Integration with AWS</strong>: Because the solution is integrated with the AWS platform, it can ingest metrics from native AWS services and resources, which can help simplify the configuration and collection of monitoring data.</li> 
 <li><strong>Scalability</strong>: AMP is highly scalable and can handle large volumes of real-time metric data, which is important for cloud environments with a lot of resources and services running.</li> 
 <li><strong>Flexibility</strong>: Prometheus is highly flexible through its Long-Term Storage (LTS) compatible with the Prometheus time series bank, which makes it integrable with tools such as Grafana.</li> 
 <li><strong>Tool Ecosystem</strong>: Prometheus is part of a broader ecosystem of cloud monitoring tools, including data visualization tools, such as Grafana, other alert tools, such as Alertmanager, and a big community that constantly creates new tools.</li> 
 <li><strong>High cardinality metrics</strong>: Each AMP environment built within a VPC can support up to 500 million active time series. This is crucial for VTEX in its need to add business rules within its metrics.</li> 
 <li><strong>Service managed by AWS</strong>: AMP is a service managed by AWS and reduces the operational load on the VTEX Observability team of having to administer services.</li> 
</ul> 
<h1>Architecture</h1> 
<p>Data collection is centralized on telemetry services installed on Amazon EKS clusters on private AWS networks. The architecture includes two forms of data collection: push and pull. Push is intended for service metrics that are sent via the OpenTelemetry Protocol. Pull is mostly aimed at collecting infrastructure metrics.</p> 
<p>In push-based data collection, the collection is done in shards, where each shard is a grouping of services that correspond to an internal division of the company’s business. This division increases the resilience of the metric system like everything else, by isolating services, which in the event of faults occur, they occur locally and not globally.</p> 
<p>In Pull-based data collection, the collection is made on services, where the grouping of services in the same Prometheus is based on the divisions of the engineering team. Each team has its own Prometheus. The filter for collecting the services is done via tags and the collection is carried out in all areas where the services are installed, thus centralizing the collection of data.</p> 
<p>Both the Prometheus and the OpenTelemetry collectors are managed by Amazon EKS. The Node Groups where the metric collectors are installed have nodes in 3 Availability Zones.</p> 
<p style="text-align: center;"><img alt="VTEX Sharding architecture for Metrics" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/03/01/arch_customer_amp.jpg" /><br /> <em>Figure 1 – Architecture for Sharding to AMP</em></p> 
<h1>Results</h1> 
<p><strong>Global Results</strong></p> 
<p>As VTEX expanded its telemetry ingestion management capacity to hundreds of services and microservices using AMP, stability has been one of its biggest benefits. One consequence of this is the increase in the resilience of monitoring services. In addition, observability management was uncomplicated by allowing focus on more important issues for VTEX, such as ingestion control, education with the use of telemetry, and improvements in the metrics pipeline. All of this directly implies an increase in reliability for VTEX engineers and their systems, in addition to improving risk and crisis management. Using AMP, VTEX gained confidence in telemetry data that allowed the company to trust, debug, and correctly observe&nbsp; systems in real time.</p> 
<p>By separating workspaces from AMP, VTEX was able to better isolate and allocate teams and services. This division minimized the failure of the VTEX Observability systems. Now, when there are faults, they occur locally and not globally. Local faults affect only a subset of the teams belonging to a workspace but never affects the entire VTEX engineering team.</p> 
<p><strong>Results in the observability team</strong></p> 
<p>Going a little deeper, we know that one of the main objectives of a reliability engineer is to automate processes and reduce toil such as repetitive and manual tasks. By moving to the fully managed Amazon Managed Service for Prometheus from Cortex, VTEX was able to focus on providing&nbsp; support to the VTEX teams, and gain control of its OpenTelemetry pipeline by using observability libraries and leveraging Grafana to visualize and query telemetry.</p> 
<p>The reliability of the services offered by the observability team increased due to the use of AMP. Using Cortex for this type of telemetry, we had a fundamental problem: unavailability for many hours. Using Cortex required management, and VTEX faced constant problems maintaining the service to keep it running. In addition, VTEX faced limitations such as limited number of time series and the problem of high cardinality of parameters.</p> 
<p><strong>Results for VTEX engineering and services</strong></p> 
<p>One of the biggest improvement points in terms of results for VTEX engineering teams is the reliability of the metrics generated by the services. VTEX teams could rely on their alarms and views based on metrics that come from AMP, not only because VTEX didn’t have unexpected interruptions in the metric telemetry service, but also because of AMP’s support for high volume capacity and&nbsp; greater cardinality of parameters in metrics.</p> 
<h1>Next Steps</h1> 
<p>This entire migration to Amazon Managed Service for Prometheus is part of a larger scope of modernizing the VTEX Observability stack. Thinking of a long-term architecture, without vendor lock-in and with flexible operation, VTEX adopted the OpenTelemetry protocol as the basic tool for this entire new architecture.</p> 
<p>Once modernized, this architecture will allow VTEX to extract information and intelligence from all &nbsp;its telemetry data such as correlation between logs, metrics and traces; anomaly detection; SLOS/SLIs; better alerts; and data enrichment.</p> 
<h1>Conclusion</h1> 
<p>In this blog, we showed how VTEX managed observability using Amazon Managed Service for Prometheus to maximize operational engineering efficiency by expanding both the volume and the quality of the metrics observed. The new architecture reduced investment in observability by 41%. Amazon Managed Service for Prometheus allowed the use of high-cardinality metrics while keeping clusters highly available, thus ensuring that VTEX engineering had visibility into their metrics.</p> 
<p>For more information, see the following references:</p> 
<ul> 
 <li><a href="https://aws-observability.github.io/observability-best-practices/guides/operational/gitops-with-amg/gitops-with-amg/">AWS Observability Best Practices Guide</a></li> 
 <li><a href="https://catalog.workshops.aws/observability/en-US/aws-managed-oss/gitops-with-amg">One Observability Workshop</a></li> 
 <li><a href="https://github.com/aws-observability/terraform-aws-observability-accelerator">Terraform AWS Observability Accelerator</a></li> 
 <li><a href="https://github.com/aws-observability/cdk-aws-observability-accelerator">CDK AWS Observability Accelerator</a></li> 
</ul> 
<p>&nbsp;</p> 
<footer> 
 <div class="blog-author-box"> 
  <h3 class="lb-h4">Gustavo Pantuza</h3> 
  <div class="blog-author-image">
   <img alt="" class="aligncenter size-full wp-image-11636" height="160" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/03/04/pantuza-300x300.jpeg" width="120" />
  </div> 
  <p>Gustavo, a former SRE engineer at VTEX, is now with Doximity. He holds a Masters in Computer Science in networks and distributed systems.</p> 
 </div> 
</footer> 
<footer> 
 <div class="blog-author-box"> 
  <h3 class="lb-h4">Rodrigo Moraes</h3> 
  <div class="blog-author-image">
   <img alt="" class="aligncenter size-full wp-image-11636" height="160" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/03/04/rodrigo-300x300.jpeg" width="120" />
  </div> 
  <p>Rodrigo, a former SRE engineer at VTEX, is a results-driven engineer with a track record of developing solutions that streamline operations and drive efficiency. He is now at Incognia.</p> 
 </div> 
</footer> 
<footer> 
 <div class="blog-author-box"> 
  <h3 class="lb-h4">Samuel D’Aquino JR</h3> 
  <div class="blog-author-image">
   <img alt="" class="aligncenter size-full wp-image-11636" height="160" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/03/04/samuel-300x300.jpeg" width="120" />
  </div> 
  <p>Samuel is an SRE engineer at VTEX and a knowledge enthusiast. He holds a computer science degree from the Federal University of Ceara and is also a technician in computer networks by E.E.E.P Gov. Luiz de Gonzaga Fonseca Mota.</p> 
 </div> 
</footer> 
<footer> 
 <div class="blog-author-box"> 
  <h3 class="lb-h4">João Borges</h3> 
  <div class="blog-author-image">
   <img alt="" class="aligncenter size-full wp-image-11636" height="160" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/04/17/joaoborges-300x267.png" width="120" />
  </div> 
  <p>João is a Technical Account Manager at AWS, working closely with a wide range of clients across various industries since 2017. His role involves providing specialized technical support, assisting customers in resolving complex challenges, optimizing their cloud architectures, and maximizing the value of AWS services. </p>
 </div> 
</footer>
