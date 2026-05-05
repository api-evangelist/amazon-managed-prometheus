---
title: "Monitoring and Visualizing Amazon EKS signals with Kiali and AWS managed open-source services"
url: "https://aws.amazon.com/blogs/mt/monitoring-and-visualizing-amazon-eks-signals-with-kiali-and-aws-managed-open-source-services/"
date: "Mon, 20 Nov 2023 19:29:17 +0000"
author: "Munish Dabra"
feed_url: "https://aws.amazon.com/blogs/mt/tag/amazon-managed-service-for-prometheus/feed/"
---
<p>Microservices architecture enables scalability and agility for modern applications. However, distributed systems can introduce complexity when troubleshooting issues across services on different machines. To gain observability into microservices environments, operators need tools to monitor, analyze, and debug the interconnected services.</p> 
<p><a href="https://istio.io/">Istio</a> service mesh connects, secures, and observes microservices communications. It provides a way to manage and monitor microservices landscapes. <a href="https://kiali.io/">Kiali</a> offers a graphical console to view and control Istio configurations by visualizing the service mesh topology, displaying metrics, and validating configurations. Kiali integrates with Prometheus to analyze the mesh and surface insights.<br /> <a href="https://aws.amazon.com/prometheus/">Amazon Managed Service for Prometheus</a> is a fully-managed monitoring and alerting service compatible with Prometheus. It allows easy monitoring of containerized applications at scale by auto-scaling ingestion, storage, alerting, and querying as workloads change.</p> 
<p>Also, we’ll explore integrating Kiali with Amazon Managed Service for Prometheus to query Prometheus metrics. We’ll also look at adding <a href="https://aws.amazon.com/grafana/">Amazon Managed Grafana</a> for advanced querying and custom Istio dashboards. This allows you to build insightful dashboards that provide visibility into the performance and health of your Istio-enabled services.</p> 
<h2><strong>Architecture</strong></h2> 
<p>The following diagram shows the architecture for deploying and configuring Kiali, Amazon Managed Service for Prometheus, <a href="https://aws.amazon.com/otel/">AWS Distro for OpenTelemetry</a> (ADOT) collector and Amazon Managed Grafana:</p> 
<p style="text-align: left;"><img alt="Figure 1 - Architecture for deploying and configuring Kiali, Amazon Managed Service for Prometheus, ADOT collector and Amazon Managed Grafana" class="wp-image-46944 size-full alignnone" height="644" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2023/11/17/figure1-2.png" width="936" /><br /> <strong>Figure 1 – Architecture for deploying and configuring Kiali, Amazon Managed Service for Prometheus, ADOT collector and Amazon Managed Grafana<br /> </strong><br /> 1. The ADOT Collector scrapes Istio telemetry metrics from the containerized application running on <a href="https://aws.amazon.com/eks/">Amazon Elastic Kubernetes Services</a> (Amazon EKS).<br /> 2. The metrics data scraped by the ADOT Collector is sent to Amazon Managed Prometheus by the ADOT’s Prometheus Remote Write Exporter.<br /> 3. Kiali visualizes the Istio telemetry metrics from Amazon Managed Service for Prometheus. Before Kiali can query the Istio metrics in Amazon Managed Service for Prometheus, it needs to authenticate with the service. Since Kiali does not natively support <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebsapis-using-sigv4.html">AWS Signature Version 4</a> (AWS Sigv4) authentication, an <a href="https://github.com/awslabs/aws-sigv4-proxy">AWS Sigv4 proxy</a> is used to enable Kiali to authenticate. AWS SigV4 is the process to add authentication information to Amazon API requests sent by HTTP header. Also, Kiali relies on the default metrics set provided by Istio, so these metrics must be available in Prometheus.<br /> 4. Kiali dashboards provide direct links to Amazon Managed Grafana for advanced analysis and queries Istio Dashboards. To enable these links, the Amazon Managed Grafana URL and preconfigured Istio dashboards must be configured in Kiali.<br /> 5. With Amazon Managed Grafana, you can leverage <a href="https://kiali.io/docs/configuration/p8s-jaeger-grafana/grafana/">querying and data transformation features</a> to customize Istio dashboard panels and create visualizations tailored to your needs. The advanced queries and customization settings enable you to fine-tune graphs and charts to focus on the most relevant Istio metrics and data for your system. When accessing Amazon Managed Grafana via the Kiali links, Amazon Managed Grafana retrieves the Istio dashboards from Amazon Managed Prometheus. And Amazon Managed Service for Prometheus is configured as data source in Amazon Managed Grafana.</p> 
<h2>Solution overview</h2> 
<h3>Prerequisites</h3> 
<p>● <a href="https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html">AWS Command Line Interface (AWS CLI)</a><br /> ● <a href="https://eksctl.io/">eksctl</a><br /> ● <a href="https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html">kubectl</a><br /> ● <a href="https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html">Create an Amazon EKS Cluster</a><br /> ● Create an <a href="https://catalog.workshops.aws/observability/en-US/aws-managed-oss/amp/setup">Amazon Managed Service for Prometheus Workspace</a><br /> ● Create an <a href="https://docs.aws.amazon.com/eks/latest/userguide/enable-iam-roles-for-service-accounts.html">IAM OIDC identity provider for your EKS cluster</a><br /> ● Install <a href="https://catalog.workshops.aws/observability/en-US/aws-managed-oss/amp/ingest-metrics">AWS Distro for OpenTelemetry (ADOT) Collector on EKS cluster</a><br /> ● Istio installation; If you don’t have Istio already installed on your EKS cluster, please refer to <a href="https://istio.io/latest/docs/setup/getting-started/">Istio Getting Started Guide</a>, ensure <a href="https://istio.io/latest/docs/setup/getting-started/#bookinfo">successful installation</a> of sample application and verify <a href="https://istio.io/latest/docs/setup/getting-started/#confirm">external access.</a></p> 
<p><strong>Step 1: Configure <a href="https://aws.amazon.com/iam">AWS Identity and Access Management</a> (IAM) role and Service Account for Kiali Proxy</strong></p> 
<p>Kiali has two main components: a back-end application running in a container, and a front-end UI application. The Kiali back-end needs access to Amazon Managed Service for Prometheus in order to query Istio metrics.</p> 
<p>To enable this access, you need to create an IAM Service Account with an IAM role that has the <strong>AmazonPrometheusQueryAccess</strong> policy attached. This IAM role should then be associated with the EKS cluster that is running Kiali. Attaching this IAM role to the EKS cluster allows the Kiali back-end pod to authenticate with Amazon Managed Service for Prometheus using the credentials from the Service Account, and query Prometheus metrics.</p> 
<p>Then, you need to enable IAM as <a href="https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_create_oidc.html">OIDC Provider</a> for EKS Cluster as it is not enabled by default. You should have done this as part of prerequisite. If already exists, below commands will not create new OIDC provider.</p> 
<p><code>export EKS_CLUSTER=&lt;Your&nbsp;EKS Cluster&nbsp;Id&gt;</code></p> 
<p><code>eksctl utils associate-iam-oidc-provider --cluster=$EKS_CLUSTER --approve</code></p> 
<p>Create new Service Account and IAM role to query Amazon Managed Service for Prometheus from Kiali.</p> 
<p><code>export AMP_SIGV4_IAM_ROLE=amp-sigv4-proxy-query-role</code></p> 
<p><code>eksctl create iamserviceaccount \</code><br /> <code>--name $AMP_SIGV4_IAM_ROLE \</code><br /> <code>--namespace istio-system \</code><br /> <code>--cluster $EKS_CLUSTER \</code><br /> <code>--attach-policy-arn arn:aws:iam::aws:policy/AmazonPrometheusQueryAccess \</code><br /> <code>--approve \</code><br /> <code>--override-existing-serviceaccounts</code></p> 
<p><strong>Step 2: Configure AWS Sigv4 authentication for querying Amazon Managed Service for Prometheus</strong></p> 
<p>AWS Sigv4 is a process to add authentication information to requests made to AWS APIs using HTTP header. The AWS CLI and the <a href="https://aws.amazon.com/tools/">AWS SDKs</a> already use this protocol to make calls to the AWS APIs. Amazon Managed Service for Prometheus requires the API calls to have sigv4 authentication, and since Kiali doesn’t support sigv4, we will be deploying a sigv4 proxy service to act as a gateway for Kiali to authenticate and access the query endpoint of the Amazon Managed Service for Prometheus.</p> 
<p>Execute the following commands to deploy the sig-v4 proxy pod on your EKS cluster:</p> 
<p><code>export AWS_REGION=us-west-2 # Update AWS region as per your use case</code></p> 
<pre><code class="lang-yaml">cat &lt;&lt; EOF &gt; kiali-sigv4.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: kiali-sigv4
  name: kiali-sigv4
  namespace: istio-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kiali-sigv4
  template:
    metadata:
      labels:
        app: kiali-sigv4
      name: kiali-sigv4
    spec:
      serviceAccountName: ${AMP_SIGV4_IAM_ROLE}
      containers:
      - name: aws-kiali-iamproxy
        image: public.ecr.aws/aws-observability/aws-sigv4-proxy:1.7
        args:
          - --name
          - aps
          - --region
          - ${AWS_REGION}
          - --host
          - aps-workspaces.${AWS_REGION}.amazonaws.com
        ports:
          - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: kiali-sigv4
  namespace: istio-system
spec:
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: kiali-sigv4
---
EOF</code><code>
</code></pre> 
<p><code># Create Kiali Sigv4 proxy pod</code><br /> <code>$ kubectl apply -f kiali-sigv4.yaml</code></p> 
<p>You should see Kiali Sigv4 proxy pod running as shown below:</p> 
<p><code>$ kubectl get pods -A | grep 'kiali-sigv4'</code></p> 
<p><code>istio-system kiali-sigv4-749dfd8694-rpt8c 1/1 Running 0 55s</code></p> 
<p><strong>Step 3: Update Kiali add-on config to point to Amazon Managed Service for Prometheus</strong></p> 
<p>Go to your Kiali add-on config under <code>/samples/addons/kiali.yaml</code> in your Istio deployment folder.</p> 
<p>By default, Kiali assumes that Prometheus is available at the URL of the form <code>http://prometheus.&lt;istio_namespace_name&gt;:9090</code>, which is the case if you are using <a href="https://istio.io/latest/docs/ops/integrations/prometheus/#option-1-quick-start">the Prometheus Istio add-on</a>. Since, in this case we are using Amazon Managed Service for Prometheus workspace to query metrics, you must manually provide the endpoint. For more information, see <a href="https://kiali.io/docs/configuration/p8s-jaeger-grafana/prometheus/">Prometheus configuration for Kiali</a>.</p> 
<p><strong>Note</strong>: If you already have any existing Kiali add-on, you can also <a href="https://kubernetes.io/docs/concepts/configuration/configmap/">edit kiali configmap</a> for the same to apply configuration changes for ‘external services’ section as discussed above and restart the kiali pod.</p> 
<p>Update the Amazon Managed Service for Prometheus URL with Workspace ID:</p> 
<p><code>WORKSPACE_ID=$(aws amp list-workspaces --alias observability-workshop | jq .workspaces[0].workspaceId -r)</code><br /> <code>AMP_URL="http://kiali-sigv4.istio-system.svc.cluster.local:80/workspaces/$WORKSPACE_ID/"</code></p> 
<p>Let’s update <strong>external services</strong> section in <code>kiali.yaml</code> file to include Amazon Managed Service for Prometheus URL. Use <code>sed</code> command to update <strong>external_services </strong>section as below. Make sure you are in your Istio deployment folder in path <code>samples/addons.</code></p> 
<p><code># AMP_URL will be updated as environmental variable</code><br /> <code>sed -i "/external_services:/a\ prometheus:\n url: \""${AMP_URL}"\"\n custom_metric_url: \""${AMP_URL}"\"\n thanos_proxy:\n enabled: true\n retention_period: "7d"\n scrape_interval: "30s" " kiali.yaml</code></p> 
<p><strong>Note</strong>: Ensure to use <a href="https://formulae.brew.sh/formula/gnu-sed">gsed</a> when running this command on macOS.</p> 
<p>The updated code snippet looks like below:</p> 
<pre><code class="lang-yaml">external_services:
      prometheus:
        url: "http://kiali-sigv4.istio-system.svc.cluster.local:80/workspaces/ws-334fc448-e889-479d-83aa-b44f01d3xxxx/"
        custom_metric_url: "http://kiali-sigv4.istio-system.svc.cluster.local:80/workspaces/ws-334fc448-e889-479d-83aa-b44f01d3xxxx/"
        thanos_proxy:
          enabled: true
          retention_period: "7d"
          scrape_interval: "30s"</code></pre> 
<p><strong>Step 4: Install Kiali add-on for Istio</strong></p> 
<p>Now, make sure you are in your Istio deployment folder and then run below to deploy Kiali add-on pod:</p> 
<p><code>$ kubectl apply -f samples/addons/kiali.yaml</code></p> 
<p><code>serviceaccount/kiali created</code><br /> <code>configmap/kiali created</code><br /> <code>clusterrole.rbac.authorization.k8s.io/kiali-viewer created</code><br /> <code>clusterrole.rbac.authorization.k8s.io/kiali created</code><br /> <code>clusterrolebinding.rbac.authorization.k8s.io/kiali created</code><br /> <code>role.rbac.authorization.k8s.io/kiali-controlplane created</code><br /> <code>rolebinding.rbac.authorization.k8s.io/kiali-controlplane created</code><br /> <code>service/kiali created</code><br /> <code>deployment.apps/kiali created</code></p> 
<p><code>$ kubectl get pods -A | grep 'kiali'</code></p> 
<p><code>istio-system kiali-5db6985fb5-h6n5z 1/1 Running 0 3m39s</code><br /> <code>istio-system kiali-sigv4-749dfd8694-rpt8c 1/1 Running 0 14m</code></p> 
<p>Let’s validate Kiali configmap to ensure configuration have been applied correctly. You should see similar to the following:</p> 
<p><code>$ kubectl describe cm kiali -n istio-system | grep url</code></p> 
<p><code>url: "http://kiali-sigv4.istio-system.svc.cluster.local:80/workspaces/ws-3ef85564-bb57-48bc-8acb- xxxxxxxxxx/"</code><br /> <code>custom_metric_url: "http://kiali-sigv4.istio-system.svc.cluster.local:80/workspaces/ws-3ef85564-bb57-48bc-8acb-xxxxxxxxxx/"</code></p> 
<p>You can validate logs to ensure there are no errors for Kiali pod:</p> 
<p><code># Validate Logs for any error</code><br /> <code>export kp=$(kubectl -n istio-system get pod -l app=kiali -o jsonpath='{.items[0].metadata.name}')</code></p> 
<p><code>kubectl logs -f $kp -n istio-system</code></p> 
<p><strong>Step 4: Generate traffic to collect telemetry data (optional)</strong></p> 
<p>To view the view telemetry data, you need to generate the traffic by accessing the application multiple times, open a new terminal tab and use these commands to send a traffic to the mesh:</p> 
<p><code>export GATEWAY_URL=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')</code></p> 
<p><code>watch --interval 1 curl -s -I -XGET "http://${GATEWAY_URL}/productpage"</code></p> 
<p>The above command sends the request for every 1 second and recommended to keep in running for at least 30 minutes to visualize the dashboard with sufficient data.</p> 
<p><strong>Step 5: Launch Kiali</strong></p> 
<p>Open a new terminal tab and launch Kiali dashboard by executing the following command</p> 
<p><code># Port Forward</code><br /> <code>export ks=$(kubectl -n istio-system get service -l app=kiali -o jsonpath='{.items[0].metadata.name}')</code><br /> <code>kubectl port-forward -n istio-system svc/$ks 20001:20001</code></p> 
<p><code>http://localhost:20001/kiali<br /> </code><br /> Optionally, if you are using <a href="https://aws.amazon.com/cloud9">AWS Cloud9</a> instance as your local environment, associate IPv4 address for port forwarding and <a href="https://docs.aws.amazon.com/cloud9/latest/user-guide/app-preview.html#app-preview-preview-app">update security rules</a> for access.</p> 
<p><code>kubectl port-forward -n istio-system svc/$ks 20001:20001 --address 0.0.0.0</code><br /> <code>http://$cloud9IP:20001/kiali</code></p> 
<p>Now, let’s take a look at the Kiali console. Kiali dashboards provide valuable insights into your service mesh’s health, including the infrastructure and application services. It enables configuration, updates, and validation of your Istio service mesh. Kiali monitors various components of the service mesh, such as <a href="https://istio.io/latest/docs/tasks/traffic-management/ingress/ingress-control/">Istio-ingressgateway</a> and <a href="https://istio.io/latest/docs/tasks/traffic-management/egress/egress-gateway/">Istio-egressgateway</a>, to identify any underlying issues. Kiali offers multiple ways to examine mesh topology, with the Kiali Graph providing a powerful visualization of mesh traffic for quick issue identification. Additionally, Kiali performs semantic validations on Istio objects, ensuring coherence across objects and even across namespaces based on the runtime status of the service mesh. Overall, Kiali dashboards are a comprehensive tool for monitoring, visualization, and validating Istio service mesh metrics.</p> 
<p>For example, in the following <strong>Workloads</strong> dashboard, you can view the <a href="https://istio.io/latest/docs/reference/config/metrics/">Istio standard metrics</a> of your application such as <strong>Request volume</strong>, <strong>Request duration</strong>, <strong>Request throughput</strong> and <strong>Response throughput</strong>.</p> 
<div class="wp-caption alignnone" id="attachment_46949" style="width: 854px;">
 <img alt="Figure 2 - Kiali Dashboard - Workloads" class="size-full wp-image-46949" height="646" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2023/11/17/figure2.jpg" width="844" />
 <p class="wp-caption-text" id="caption-attachment-46949"><strong>Figure 2 – Kiali Dashboard – Workloads</strong></p>
</div> 
<p><strong>Note</strong>: If you have installed your sample application bookinfo in default namespace select default namespace to view workload metrics.</p> 
<p><strong>Step 6: (Optional) Kiali Integration with Amazon Managed Grafana</strong></p> 
<p><a href="https://aws.amazon.com/grafana/">Amazon Managed Grafana</a> enables you to instantly query, correlate, and visualize operational metrics, logs, and traces for their applications from multiple data sources. You can integrate Amazon Managed Grafana with Kiali if you are looking for advanced querying options and highly customizable settings for Istio dashboards.</p> 
<p>Refer to <a href="https://aws.amazon.com/blogs/mt/amazon-managed-grafana-getting-started/">Amazon Managed Grafana – Getting Started</a> for information on how to configure and setup Amazon Managed Grafana. Amazon Managed Grafana lets you to configure user access through AWS IAM Identity Center or other <a href="https://aws.amazon.com/blogs/mt/amazon-managed-grafana-supports-direct-saml-integration-with-identity-providers/">SAML based Identity Providers (IdP)</a>. In this post, we’re using the <a href="https://aws.amazon.com/single-sign-on/">AWS IAM Identity Center</a> option with Amazon Managed Grafana. To set up Authentication and Authorization, follow the instructions in the Amazon Managed Grafana User Guide for enabling AWS IAM Identity Center. Refer <a href="https://aws.amazon.com/blogs/mt/monitor-istio-on-eks-using-amazon-managed-prometheus-and-amazon-managed-grafana/">Monitor Istio blog post</a> to import Istio dashboards and query metrics from Amazon Managed service Prometheus workspace.</p> 
<p>Kiali relies on<a href="https://istio.io/latest/docs/reference/config/metrics/"> Istio’s default being metrics</a> set. Therefore, it is recommended to add Istio Dashboards in the Import via grafana.com text box in Amazon Managed Grafana. A few curated Istio dashboards you can leverage are – <a href="https://grafana.com/grafana/dashboards/7636-istio-service-dashboard/">Service Dashboard</a>, <a href="https://grafana.com/grafana/dashboards/7630-istio-workload-dashboard/">Workload Dashboard</a>, <a href="https://grafana.com/grafana/dashboards/7639-istio-mesh-dashboard/">Mesh Dashboard</a>, <a href="https://grafana.com/grafana/dashboards/7645-istio-control-plane-dashboard/">Control Plane Dashboard</a>, <a href="https://grafana.com/grafana/dashboards/11829-istio-performance-dashboard/">Performance Dashboard</a> and <a href="https://grafana.com/grafana/dashboards/13277-istio-wasm-extension-dashboard/">Wasm Extension Dashboard</a>.</p> 
<p>Since you have already updated Kiali add-on in step 3, now update the configuration for adding Grafana link in Kiali dashboard in the file <code>/samples/addons/kiali.yaml</code> in your Istio deployment folder on EKS cluster.</p> 
<p>Kiali uses <a href="https://kiali.io/docs/configuration/kialis.kiali.io/#.spec.external_services.grafana.auth.type">Amazon Grafana API Key as token</a> to access Grafana. Let’s first get the Grafana API key from Amazon Managed Grafana workspace.</p> 
<p><code>$ export GRAFANA_API_KEY=$(aws grafana create-workspace-api-key --key-name "kiali_access_key3"&nbsp;\&nbsp;</code><br /> <code>&nbsp; &nbsp; &nbsp; &nbsp;&nbsp;--key-role "ADMIN" --seconds-to-live 432000&nbsp; \&nbsp;</code><br /> <code>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;--workspace-id "<strong>&lt;Grafana workspace_id&gt;</strong>" | jq '.key')&nbsp;# Update with your Grafana workspace ID</code><br /> <code>$echo $GRAFANA_API_KEY</code></p> 
<p>Now, update the <strong>Grafana_API_key</strong> from preceding and update your <strong>Grafana_workspace_URL. </strong>Update <code>kiali.yaml</code> as shown in the following code block. For more information on the detailed Kiali configuration for Grafana link refer <a href="https://kiali.io/docs/configuration/p8s-jaeger-grafana/grafana/">Kiali documentation</a>.<code></code></p> 
<pre><code class="lang-yaml"># kiali.yaml
external_services:
      custom_dashboards:
        enabled: true
      grafana:
        auth:
          type: "bearer"
          token: "&lt;Grafana_API_key&gt;" # update Grafana API Key 
        enabled: true
        in_cluster_url: "&lt;Grafana_workspace_URL&gt;" # Update Grafana Workspace URL 
        url: "&lt;Grafana_workspace_URL&gt;"
        dashboards:
        - name: "Istio Service Dashboard"
          variables:
            namespace: "var-namespace"
            service: "var-service"
        - name: "Istio Workload Dashboard"
          variables:
            namespace: "var-namespace"
            workload: "var-workload"
        - name: "Istio Mesh Dashboard"
        - name: "Istio Control Plane Dashboard"
        - name: "Istio Performance Dashboard"
        - name: "Istio Wasm Extension Dashboard"
</code></pre> 
<p>After updating the <code>kiali.yaml</code> , re-deploy the Kiali add-on for Istio.</p> 
<p><code>$ kubectl apply -f samples/addons/kiali.yaml</code></p> 
<p><code>serviceaccount/kiali created</code><br /> <code>configmap/kiali created</code><br /> <code>clusterrole.rbac.authorization.k8s.io/kiali-viewer created</code><br /> <code>clusterrole.rbac.authorization.k8s.io/kiali created</code><br /> <code>clusterrolebinding.rbac.authorization.k8s.io/kiali created</code><br /> <code>role.rbac.authorization.k8s.io/kiali-controlplane created</code><br /> <code>rolebinding.rbac.authorization.k8s.io/kiali-controlplane created</code><br /> <code>service/kiali created</code><br /> <code>deployment.apps/kiali created</code></p> 
<p>Now, navigate to Kiali Dashboard as explained in Step 5, you should be able to view clickable link as <strong>View in Grafana</strong> in <strong>Workloads &gt; Outbound Metrics</strong> tab.</p> 
<p><img alt="" class="size-full wp-image-46958 alignnone" height="272" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2023/11/17/figure3.jpg" width="844" /><br /> <strong>Figure 3 – Kiali Dashboard with Grafana link</strong></p> 
<p>After clicking on Grafana link, you will be redirected to Amazon Managed Grafana Dashboard (Istio Workload Dashboard) which looks like below:</p> 
<p><img alt="Figure 4 - Amazon Managed Grafana Dashboard (Request Volume, Request Duration)" class="size-full wp-image-46961 alignnone" height="498" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2023/11/17/figure4.jpg" width="844" /><br /> <strong>Figure 4 – Amazon Managed Grafana Dashboard (Request Volume, Request Duration)</strong></p> 
<p>You can view the same Istio metrics on both Kiali dashboard (below) as well as Amazon Managed Grafana. You notice that <code>Request Volume</code> around 0.75-1 ops/sec and Request Duration (Percentile 50) around 3ms.</p> 
<p><img alt="Figure 5 - Kiali Dashboard (Request Count, p50)" class="size-full wp-image-46964 alignnone" height="532" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2023/11/17/figure5.jpg" width="844" /><br /> <strong>Figure 5 – Kiali Dashboard (Request Count, p50)</strong></p> 
<p>For example, from Kiali <strong>Workloads</strong> dashboard your Grafana link will take you to ‘Istio Workload Dashboard’ and similarly <strong>Services</strong> to ‘Istio Service Dashboard’. You can view General, Inbound workloads and Outbound Services <a href="https://istio.io/latest/docs/tasks/observability/metrics/using-istio-dashboard/">Istio metrics</a> of workloads and services in Amazon Managed Grafana dashboard.</p> 
<h3>Recommendations for Prometheus Resource Optimization</h3> 
<h4>Metric Thinning</h4> 
<p>Istio and Envoy generate a large amount of telemetry for analysis and troubleshooting. It can result in significant resources required to ingest and store the telemetry and to support queries into the data. If you use the telemetry specifically to support Kiali, you can drop unnecessary metrics and labels on the required metrics.</p> 
<p>Applying <code>metric_relabel_configs</code> in Prometheus configuration should reduce the number of stored metrics by about 20% and the number of attributes stored on many remaining metrics.</p> 
<h4>Scrape Interval</h4> 
<p>The Prometheus <code>globalScrapeInterval</code> is a vital configuration <a href="https://kiali.io/docs/configuration/p8s-jaeger-grafana/prometheus/#scrape-interval">option</a>. The scrape interval can have a significant effect on metrics collection overhead. It affects storage for the data points and impacts query results (the more data points, the more processing and aggregation). Users should think carefully about their configured scrape interval. Note that the Istio addon for Prometheus configures it to 15s. It is suitable for demos but may be too frequent for production scenarios. The recommendation for Kiali is to set the longest interval while still providing a useful granularity. The longer the interval, the fewer data points scraped, thus reducing processing, storage, and computational overhead. But the impact on Kiali should be understood. It is essential to realize that request rates (or byte rates, message rates, etc.) require at least two data points. For Kiali to show anything useful in the graph or anywhere rates are used (many places), the minimum duration must be <strong>&gt;= 2 x globalScrapeInterval</strong>.</p> 
<h2>Clean-up</h2> 
<p>Use the following commands to clean up the created AWS resources for this demonstration.</p> 
<p><code># Delete Amazon Managed Grafana </code><br /> <code>aws grafana delete-workspace --workspace-id &nbsp;$AMG_WORKSPACE_ID</code></p> 
<p><code># Delete Amazon Managed Prometheus</code><br /> <code>export&nbsp;AMP_WORKSPACE_ID=$(aws amp list-workspaces --alias observability-workshop |&nbsp;jq .workspaces[0].workspaceId -r)</code><br /> <code>aws amp delete-workspace --workspace-id &nbsp;$AMP_WORKSPACE_ID</code></p> 
<p><code># Uninstall ADOT Collector</code><br /> <code>kubectl delete -f ./otel-collector-prometheus.yaml</code></p> 
<p><code># Cleaning up bookinfo application and istio</code><br /> <code>kubectl -n bookinfo delete -f ./samples/bookinfo/networking/bookinfo-gateway.yaml </code><br /> <code>kubectl -n bookinfo delete -f ./samples/bookinfo/platform/kube/bookinfo.yaml </code><br /> <code>kubectl delete&nbsp;ns istio-system</code></p> 
<p><code># Delete EKS Cluster</code><br /> <code>eksctl delete cluster --name &lt;YOUR_EKS_CLUSTER&gt;&nbsp;--region &lt;AWS_REGION&gt;</code></p> 
<h2>Conclusion</h2> 
<p>Kiali dashboards provide visibility into Istio service mesh metrics and can be used to monitor, visualize, and validate your application. This blog post demonstrates how to set up and access the Kiali dashboard to query Istio metrics in Amazon managed Prometheus. It also shows how to integrate Kiali with Amazon Managed Grafana to build customized dashboards and visualize metrics. Best practices are provided for optimizing metric usage, including reducing the number of stored metrics and adjusting the scrape interval. Overall, the post is a step-by-step guide to using Kiali, Amazon Managed Service for Prometheus and Amazon Managed Grafana to gain visibility into Istio service mesh metrics and build effective monitoring dashboards.</p> 
<p>To learn more AWS observability services, check out <a href="https://catalog.us-east-1.prod.workshops.aws/workshops/31676d37-bbe9-4992-9cd1-ceae13c5116c/en-US/intro">One Observability Workshop</a>. This workshop provides hands-on experience with <a href="https://aws.amazon.com/pm/cloudwatch/">Amazon CloudWatch</a>, <a href="https://aws.amazon.com/xray/">AWS X-Ray</a>, Amazon Managed Service for Prometheus, Amazon Managed Grafana, and AWS Distro for OpenTelemetry (ADOT).</p> 
<p><strong>About the authors:</strong></p> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="aligncenter size-full wp-image-11636" height="160" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2023/01/15/mundabra.jpg" width="120" />
  </div> 
  <h3 class="lb-h4">Munish Dabra</h3> 
  <p>Munish Dabra is a Principal Solutions Architect at Amazon Web Services (AWS). His current areas of focus are AI/ML and Observability. He has a strong background in designing and building scalable distributed systems. He enjoys helping customers innovate and transform their business in AWS. LinkedIn: /mdabra</p> 
 </div> 
</footer> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="aligncenter size-full wp-image-11636" height="160" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2023/11/17/vinod-singh.png" width="120" />
  </div> 
  <h3 class="lb-h4">Vinod Singh</h3> 
  <p>Vinod Singh is a Sr. Solutions Architect at Amazon Web Services. Vinod has been in industry for more than two decades in Software Architecture, Design and Development, with the last seven years in a customer facing role helping customers adopt cloud technologies. Before joining AWS, he was with Microsoft and IBM. Vinod has co-authored one of the best books on Docker. His current interests are in Containers and AI/ML. Vinod is based out of Austin, Texas and supports mentoring folks for Cloud technologies.</p> 
 </div> 
</footer> 
<footer> 
 <div class="blog-author-box"> 
  <div class="blog-author-image">
   <img alt="" class="aligncenter size-full wp-image-11636" height="160" src="https://d2908q01vomqb2.cloudfront.net/972a67c48192728a34979d9a35164c1295401b71/2022/11/22/Abhi-Khanna-Profile.jpg" width="120" />
  </div> 
  <h3 class="lb-h4">Abhi Khanna</h3> 
  <p>Abhi Khanna is a Senior Product Manager at AWS specializing in Amazon Managed Service for Prometheus. He has been involved with Observability products for the last 3 years, helping customers build towards more perfect visibility. He enjoys helping customers simplify their monitoring experience. His interests include software engineering, product management, and building things.</p> 
 </div> 
</footer>
