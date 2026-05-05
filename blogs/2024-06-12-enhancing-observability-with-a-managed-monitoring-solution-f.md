---
title: "Enhancing observability with a managed monitoring solution for Amazon EKS"
url: "https://aws.amazon.com/blogs/mt/enhancing-observability-with-a-managed-monitoring-solution-for-amazon-eks/"
date: "Wed, 12 Jun 2024 00:32:26 +0000"
author: "Siva Guruvareddiar"
feed_url: "https://aws.amazon.com/blogs/mt/tag/amazon-managed-service-for-prometheus/feed/"
---
<h2>Introduction</h2> 
<p>Keeping a watchful eye on your Kubernetes infrastructure is crucial for ensuring optimal performance, identifying bottlenecks, and troubleshooting issues promptly. In the ever-evolving world of cloud-native applications, Amazon Elastic Kubernetes Service (<a href="https://aws.amazon.com/eks/">EKS</a>) has emerged as a popular choice for deploying and managing containerized workloads. However, monitoring Kubernetes clusters can be challenging due to their complexity and AWS recently launched Amazon <a href="https://aws.amazon.com/blogs/mt/new-container-insights-with-enhanced-observability-for-amazon-eks/">CloudWatch Container Insights</a> to simplify the process. Imagine having a monitoring solution tailored specifically for your EKS clusters using Open Source, delivering real-time insights into the health and performance of your Kubernetes environment. With this, users can monitor a Kubernetes cluster’s real-time state to quickly identify issues or bottlenecks, spotting problems like memory leaks in individual containers through container-level metrics and visually analyzing across different cluster layers. With the combined power of both <a href="https://aws.amazon.com/grafana">Amazon Managed Grafana</a>&nbsp; and <a href="https://aws.amazon.com/prometheus/">Amazon Managed Service for Prometheus,</a> you can now deploy an <a href="https://docs.aws.amazon.com/grafana/latest/userguide/solution-eks.html">AWS-supported&nbsp;solution for monitoring EKS infrastructure</a>.</p> 
<p>With this solution, you can deploy a fully-managed Prometheus backend to collect and store metrics from your EKS cluster, while leveraging the intuitive visualization capabilities of Amazon Managed Grafana. A set of preconfigured dashboards will provide you with a holistic view of the health, performance, and resource utilization of your cluster. Whether you’re managing a small development cluster or a large-scale production environment, these dashboards offer better insights. From assessing the overall cluster health to monitoring the control and data planes, you’ll have a comprehensive understanding of your Kubernetes ecosystem. Additionally, you can dive deeper into workload performance across namespaces, track resource usage (CPU, memory, disk, and network), and identify potential bottlenecks before they escalate. In the following sections, we’ll explore the power of this AWS-managed solution, guiding you through the process of deploying and utilizing the pre-built CloudFormation template. Get ready to unlock a new level of visibility and control over your Amazon EKS infrastructure, empowering you to make informed decisions and optimize your Kubernetes environment for optimal performance.</p> 
<h2>Prerequisites</h2> 
<p>You will need the following resources and tools to deploy the solution:</p> 
<ul> 
 <li><a href="https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html">AWS Command Line Interface (AWS CLI) version 2</a></li> 
 <li><a href="https://github.com/weaveworks/eksctl#installation">eksctl</a></li> 
 <li><a href="https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html">kubectl</a></li> 
 <li><a href="https://helm.sh/">Helm</a></li> 
 <li><a href="https://stedolan.github.io/jq/download/">jq</a></li> 
 <li><a href="http://github.com/">git</a></li> 
 <li><a href="https://aws.amazon.com/prometheus/">Amazon Managed Service for Prometheus</a></li> 
 <li><a href="https://aws.amazon.com/grafana/">Amazon Managed Grafana</a></li> 
</ul> 
<h2>Solution Overview</h2> 
<p>This AWS-managed solution offers a comprehensive monitoring framework for your Amazon Elastic Kubernetes Service (EKS) clusters.&nbsp; The solution empowers you with anticipatory capabilities, enabling you to drive intelligent scheduling decisions based on historical usage tracking, plan for future resource demands by analyzing current utilization data, and identify potential issues early by monitoring resource consumption trends at the namespace level. On the corrective front, you can quickly troubleshoot and reduce mean time to detection (MTTD) of issues across infrastructure and workloads using the pre-configured troubleshooting dashboard. With this AWS-managed solution tailored for Amazon EKS clusters, you gain monitoring and observability capabilities. Stay ahead of performance bottlenecks, optimize resource utilization, and maintain a healthy and efficient Kubernetes environment through deep insights into your cluster’s health, performance, and resource usage.</p> 
<p>To use this solution, we need to have an EKS cluster, Amazon Managed Service for Prometheus workspace and Amazon Managed Grafana workspace. First four steps below covers setting up of these prerequisites. Then we deploy the cloud formation stack to deploy the solution and visualize the results. Finally we see the cost involved and the cleanup section.</p> 
<p style="text-align: center;"><img alt="" class="alignnone wp-image-52108 size-full" height="474" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/05/13/5img.png" width="689" /><br /> Fig 1. Data Flow diagram</p> 
<h3>Step 1: Setup the environment variables and artifacts</h3> 
<div class="hide-language"> 
 <div class="hide-language"> 
  <pre><code class="lang-bash">export CLUSTER_NAME=eks-cluster 
export AWS_REGION=&lt;REGION&gt; 
export ACCOUNT_ID=`aws sts get-caller-identity |jq -r ".Account"`</code></pre> 
 </div> 
</div> 
<h3>Step 2: Create an Amazon EKS Cluster</h3> 
<p>An Amazon EKS cluster can be created using the eksctl command line tool which provides a simple way to get started for basic cluster creation with sensible defaults as below.</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">cat &lt;&lt; EOF &gt; eks-cluster-config.yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
  name: eks-cluster
  region: us-west-2
  version: "1.29"
iam:
  withOIDC: true  
managedNodeGroups:
  - name: managed-ng
    minSize: 1
    maxSize: 2
accessConfig:
  authenticationMode: API_AND_CONFIG_MAP
iamIdentityMappings:
  - arn: arn:aws:iam::${ACCOUNT_ID}:role/Administrator
    groups:
      - system:masters
    username: admin
    noDuplicateARNs: true # prevents shadowing of ARNs
vpc:
  clusterEndpoints:
    privateAccess: true
    publicAccess: true    
EOF

eksctl create cluster -f eks-cluster-config.yaml</code></pre> 
</div> 
<p>Lets create an IAM role with access to the cluster and store the results with environment variables.</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">cat &lt;&lt; EOF &gt; trust-policy.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "eks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    },
    {
      "Sid": "Statement1",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::${ACCOUNT_ID}:root"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF
aws iam create-role --role-name EKSClusterAdminRole --assume-role-policy-document file://trust-policy.jsonROLE_ARN=$(aws iam get-role --role-name EKSClusterAdminRole --query Role.Arn --output text)
VPC_ID=$(aws eks describe-cluster --name $CLUSTER_NAME --query cluster.resourcesVpcConfig.vpcId --output text --region $AWS_REGION)
SECURITY_GROUP_ID=$(aws eks describe-cluster --name $CLUSTER_NAME --query cluster.resourcesVpcConfig.clusterSecurityGroupId --output text --region $AWS_REGION)
SUBNET_IDS=$(aws eks describe-cluster --name $CLUSTER_NAME --query cluster.resourcesVpcConfig.subnetIds --output text --region $AWS_REGION)
OIDC_URL=$(aws eks describe-cluster --name $CLUSTER_NAME --query "cluster.identity.oidc.issuer" --output text --region $AWS_REGION)</code></pre> 
</div> 
<p>Lets create an access entry for the above IAM role, and give the EKSClusterAdmin access</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">aws eks create-access-entry --cluster-name $CLUSTER_NAME --principal-arn $ROLE_ARN --region $AWS_REGION
aws eks associate-access-policy --cluster-name $CLUSTER_NAME --principal-arn $ROLE_ARN --access-scope type=cluster --policy-arn arn:aws:eks::aws:cluster-access-policy/AmazonEKSClusterAdminPolicy --region $AWS_REGION</code></pre> 
</div> 
<h3>Step 3: Create Amazon Managed Service for Prometheus Workspace</h3> 
<p>The ‘<em>aws amp create-workspace</em>‘ command creates an Amazon Managed Service for Prometheus workspace with the alias ‘<em>AMP-EKS</em>‘ in the specified AWS region. The workspaces provide isolated environments for storing Prometheus metrics and dashboards. The workspace is created with default settings which can be further customized if needed. The call returns the ID of the newly created workspace. This ID is required for sending metrics data to the workspace from applications as well as for allowing other services to access the data.</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">aws amp create-workspace --alias AMP-EKS --region $AWS_REGION
AMP_WS_ID=$(aws amp list-workspaces --region $AWS_REGION| jq -r ".workspaces[0].workspaceId")</code></pre> 
</div> 
<h3>Step 4: Create Amazon Managed Grafana workspace</h3> 
<p>Create an Amazon Managed workspace compatible with Grafana version 9 by following the instructions <a href="https://docs.aws.amazon.com/grafana/latest/userguide/AMG-create-workspace.html">here</a>. Also you can choose to assign users as “admin” to the workspace. Lets get the Grafana workspace ID using the below command</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">AMG_WS_ID=$(aws grafana list-workspaces --region $AWS_REGION --query "workspaces[0].id" --output text)</code></pre> 
</div> 
<p>Create an API Key with ADMIN access for calling Grafana HTTP APIs using <a href="https://docs.aws.amazon.com/grafana/latest/userguide/using-api-keys.html">these</a> instructions and store it in AMG_API_KEY variable. Store the parameter in the Systems Manager parameter store as below</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">aws ssm put-parameter --name "/eks-infra-monitoring-accelerator/grafana-api-key" \
    --type "SecureString" \
    --value $AMG_API_KEY \
    --region $AWS_REGION</code></pre> 
</div> 
<h3>Step 5: Deploy the solution using CloudFormation</h3> 
<p>Create an S3 bucket, get the solution files from the GitHub repo and upload to S3 using the below commands:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash"> aws s3 mb s3://&lt;s3-bucket&gt;
 git clone --no-checkout https://github.com/aws-observability/observability-best-practices.git
 cd observability-best-practices
 git sparse-checkout init --cone
 git sparse-checkout set solutions/oss/eks-infra/v1.0.0/iac
 aws s3 cp solutions/oss/eks-infra/v1.0.0/iac s3://&lt;s3-bucket&gt; --recursive</code></pre> 
</div> 
<p>Uploaded files from S3 looks like below.Note the URL of eks-monitoring-cfn-template.json as we will need this in the next steps.</p> 
<p><img alt="Fig 2. S3 bucket showing the Solution files" class="aligncenter wp-image-52090 size-full" height="1178" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/05/13/image2-1.png" width="1418" /></p> 
<p style="text-align: center;">Fig 2. S3 bucket showing the Solution files</p> 
<p>You can provision the solution using CloudFormation via the CLI like so:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">aws cloudformation deploy --stack-name amg-solution \
    --region $AWS_REGION \
    --capabilities CAPABILITY_IAM \ 
    --template-file https://amg-s3-bucket.s3.us-east-2.amazonaws.com/eks-monitoring-cfn-template.json \
    --parameter-overrides \
    AMGWorkspaceEndpoint=$AMG_WS_ID.grafana-workspace.$AWS_REGION.amazonaws.com \
    AMPWorkspaceId=$AMP_WS_ID \
    EKSClusterAdminRoleARN=$ROLE_ARN \
    EKSClusterName=$CLUSTER_NAME \
    EKSClusterOIDCEndpoint=$OIDC_URL \
    EKSClusterSecurityGroupId=$SECURITY_GROUP_ID \
    EKSClusterSubnetIds=$SUBNET_IDS \
    EKSClusterVpcId=$VPC_ID \
    S3BucketName=amg-s3-bucket \
    S3BucketRegion=$AWS_REGION </code></pre> 
</div> 
<p>The other option is to use the AWS Console and go to CloudFormation → Create Stack and enter the values like below, providing the values to create the resources:</p> 
<p><img alt="Fig 3. CloudFormation screen showing sample values" class="aligncenter wp-image-52091 size-full" height="1364" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/05/13/image3-1.png" width="1462" /></p> 
<p style="text-align: center;">Fig 3. CloudFormation screen showing sample values</p> 
<p>Creating the stack take around 20 minutes to complete. After the stack creation is complete, you must configure the Amazon EKS cluster to allow access from the newly created scraper. You can get the Scraper ID from your EKS cluster’s Observability tab. Use this ID, and follow these <a href="https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-collector-how-to.html#AMP-collector-eks-setup">instructions</a> to configure your Amazon EKS cluster for managed scraping.</p> 
<h3>Step 6: Solution overview</h3> 
<p>Once the steps the completed, log into your Amazon Managed Grafana workspace and under Dashboards, you should be able to view various dashboards under “EKS Infrastructure Monitoring” as below. This has both Infrastructure as well as workload related dashboards.</p> 
<p><img alt="Fig 4. Amazon Managed Grafana dashboards" class="aligncenter wp-image-52092 size-full" height="1480" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/05/13/image-6.png" width="2704" /></p> 
<p style="text-align: center;">Fig 4. Amazon Managed Grafana dashboards</p> 
<p>The Cluster dashboard under Computer Resources shows the various metrics related to the cluster as below. As you can see the CPU utilization is low since not much workloads are running</p> 
<p><img alt="Fig 5. Amazon Managed Grafana Dashboard showing Cluster view" class="aligncenter wp-image-52093 size-full" height="1372" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/05/13/image-7.png" width="2694" /></p> 
<p style="text-align: center;">Fig 5. Amazon Managed Grafana Dashboard showing Cluster view</p> 
<p>The Namespace(workload) dashboard provides similar information. You can thinks of this as parallel to what you might be viewing from the CloudWatch Container Insight’s Namespace view.</p> 
<p><img alt="Fig 6. Amazon Managed Grafana Dashboard showing Namespace view" class="aligncenter wp-image-52094 size-full" height="980" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/05/13/image-8.png" width="2712" /></p> 
<p style="text-align: center;">Fig 6. Amazon Managed Grafana Dashboard showing Namespace view</p> 
<p>Same is the case with workload view</p> 
<p><img alt="Fig 7. Amazon Managed Grafana Dashboard showing workspaces view" class="aligncenter wp-image-52095 size-full" height="538" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/05/13/image-9.png" width="2714" /></p> 
<p style="text-align: center;">Fig 7. Amazon Managed Grafana Dashboard showing workloads view</p> 
<p>You will also get Control plane views as well like below with the kube-apiserver view, which shows the advanced kubeapi-server metrics</p> 
<p><img alt="Fig 8. Amazon Managed Grafana Dashboard showing advanced kube-apiserver view" class="aligncenter wp-image-52096 size-full" height="1370" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/05/13/image-10-1.png" width="2704" /></p> 
<p style="text-align: center;">Fig 8. Amazon Managed Grafana Dashboard showing advanced kube-apiserver view</p> 
<p>Also you will be getting the Kube-apiserver troubleshooting view as well like below, which will be helpful during the troubleshooting activities for your cluster.</p> 
<p><img alt="Fig 9. Amazon Managed Grafana Dashboard showing troubleshooting kube-apiserver view" class="aligncenter wp-image-52097 size-full" height="976" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/05/13/image-11-1.png" width="2718" /></p> 
<p style="text-align: center;">Fig 9. Amazon Managed Grafana Dashboard showing troubleshooting kube-apiserver view</p> 
<p>Also the kubelet dashboard view as well</p> 
<p><img alt="Fig 10. Amazon Managed Grafana Dashboard showing Kubelet view" class="aligncenter wp-image-52098 size-full" height="802" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/05/13/image-12-1.png" width="2270" /></p> 
<p style="text-align: center;">Fig 10. Amazon Managed Grafana Dashboard showing Kubelet view</p> 
<p>And last but not least, Node dashboard view looks like below which shows the CPU and load average. Again since not much workloads are running now, the charts does not show lot of variation. These various dashboards tracks a total of 88 metrics and the full list of metrics is documented <a href="https://docs.aws.amazon.com/grafana/latest/userguide/solution-eks.html">here</a>.</p> 
<p><img alt="Fig 11. Amazon Managed Grafana Dashboard showing Nodes view" class="aligncenter wp-image-52099 size-full" height="394" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/05/13/image-13-1.png" width="1859" /></p> 
<p style="text-align: center;">Fig 11. Amazon Managed Grafana Dashboard showing Nodes view</p> 
<h3>Using the solution for performance monitoring</h3> 
<p>Let us deploy some workload and load test to see the anticipatory capabilities. For this, we launch a Java application consisting of a Kubernetes Deployment and Service, using the Amazon Correto JDK:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">kubectl create deployment sample-deploy --image=public.ecr.aws/m8u2y2m7/gravitonjava:vcorreto --replicas=3
deployment.apps/sample-deploy created
kubectl expose deployment sample-deploy --name=sample-svc --port=8080 --target-port=8080 --type=ClusterIP
service/sample-svc exposed</code></pre> 
</div> 
<p>Now let us stress-test this deployment using the <a href="https://github.com/aws-observability/cdk-aws-observability-accelerator">wrk</a> tool as below. This will spin up 64 threads creating 2,048 connections for a period of 15 minutes, targeting the service we created in the previous step.</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">cat &lt;&lt; EOF &gt; job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: wrk-job
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: wrk
        image: ruslanys/wrk:ubuntu
        command: ["/usr/local/bin/wrk", "-t64", "-c2048", "-d900s", "http://sample-svc.default.svc.cluster.local:8080"]
EOF
kubectl apply -f job.yaml</code></pre> 
</div> 
<p>After this, you should be able to see the CPU and Load average spiking up as below from the Nodes dashboard</p> 
<p><img alt="Fig 12. Amazon Managed Grafana Dashboard Node view with CPU utilization" class="aligncenter wp-image-52100 size-full" height="403" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/05/13/image-14-1.png" width="1866" /></p> 
<p style="text-align: center;">Fig 12. Amazon Managed Grafana Dashboard Node view with CPU utilization</p> 
<p>Same way, from the Cluster dashboard also we should be able to see the hight CPU utilization as below.</p> 
<p><img alt="Fig 13. Amazon Managed Grafana Dashboard Cluster view with CPU utilization" class="aligncenter wp-image-52101 size-full" height="284" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2024/05/13/image-15-1.png" width="1864" /></p> 
<p style="text-align: center;">Fig 13. Amazon Managed Grafana Dashboard Cluster view with CPU utilization</p> 
<h2>Cleanup</h2> 
<p>Use the following commands to delete resources created during this post:</p> 
<div class="hide-language"> 
 <pre><code class="lang-bash">aws grafana delete-workspace --workspace-id $AMG_WS_ID
aws iam delete-role --role-name $ROLE_ARN
aws amp delete-workspace --workspace-id $AMP_WS_ID
eksctl delete cluster $EKS_CLUSTER</code></pre> 
</div> 
<h2>Costs</h2> 
<p>This solution leverages AWS managed services, including Amazon Managed Grafana and Amazon Managed Service for Prometheus, to provide comprehensive monitoring and observability for your Amazon EKS clusters. While these services offer convenience and ease of use, it’s important to note that you will incur standard usage charges. These charges include costs associated with Amazon Managed Grafana workspace access by users, as well as metric ingestion and storage within Amazon Managed Service for Prometheus. The number of metrics ingested, and consequently the associated costs, will depend on the configuration and usage of your Amazon EKS cluster. You can monitor the ingestion and storage metrics through CloudWatch, as detailed in the Amazon Managed Service for Prometheus User Guide. Additionally, AWS provides a <a href="https://calculator.aws/">pricing calculator</a> to help estimate the costs based on the number of nodes in your EKS cluster, which directly impacts the metric ingestion volume.</p> 
<h2>Conclusion</h2> 
<p>The AWS-managed solution for monitoring Amazon EKS clusters with Amazon Managed Grafana and Amazon Managed Service for Prometheus offers a comprehensive and streamlined approach to gaining deep insights into your Kubernetes infrastructure. By leveraging pre-configured dashboards and automated metric collection, you can effortlessly monitor the health and performance of your control and data planes, workloads, and resource utilization across namespaces. This solution empowers you with both anticipatory and corrective capabilities, enabling you to stay ahead of potential issues, optimize resource allocation, and troubleshoot problems quickly and effectively.</p> 
<p>Throughout this walkthrough, you’ve learned how to set up the necessary components, including an EKS cluster, Managed Prometheus workspace, and Managed Grafana workspace. You’ve also deployed the CloudFormation template, which orchestrates the integration of these services, providing you with a unified monitoring solution tailored for your Amazon EKS environment. With the ability to visualize and analyze a wide range of metrics, from cluster-level metrics to workload-specific insights, you can make informed decisions, ensure optimal performance, and maintain a healthy and efficient Kubernetes ecosystem.</p> 
<p>We’re looking forward to hear from you about how we can improve this solution. For example, by adding support for logs, alerts, traces, monitoring a fleet of EKS clusters, correlating telemetry, additional ways to provision the solution (for example, Terraform), and really anything else that comes to mind.</p> 
<p>To learn more about AWS Observability, see the following references:<br /> • <a href="https://aws-observability.github.io/observability-best-practices/guides/operational/gitops-with-amg/gitops-with-amg/">AWS Observability Best Practices Guide</a><br /> • <a href="https://catalog.workshops.aws/observability/en-US/aws-managed-oss/gitops-with-amg">One Observability Workshop</a><br /> • <a href="https://github.com/aws-observability/terraform-aws-observability-accelerator">Terraform AWS Observability Accelerator</a><br /> • <a href="https://github.com/aws-observability/cdk-aws-observability-accelerator">CDK AWS Observability Accelerator</a></p>
