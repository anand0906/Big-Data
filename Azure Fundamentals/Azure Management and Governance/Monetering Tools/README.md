<h1>Azure Advicer</h1>
<p><strong>Azure Advisor: An Overview</strong></p>

<p>Azure Advisor is a tool that helps you optimize your Azure resources. It provides personalized recommendations to improve various aspects of your Azure environment. The goal is to help you make your cloud setup more reliable, secure, performant, cost-effective, and operationally excellent.</p>

<p><strong>Key Areas of Recommendations</strong></p>

<ul>
    <li><strong>Reliability</strong></li>
    <li><strong>Security</strong></li>
    <li><strong>Performance</strong></li>
    <li><strong>Operational Excellence</strong></li>
    <li><strong>Cost</strong></li>
</ul>

<ol>
    <li><strong>Reliability</strong></li>
    <p><strong>Purpose:</strong> Ensures that your applications and services remain available and continue to function properly.</p>
    <p><strong>Example:</strong></p>
    <ul>
        <li><strong>Scenario:</strong> You have a web application running on Azure. Azure Advisor might suggest enabling geo-replication for your virtual machines (VMs) to ensure that if one region goes down, your application remains available in another region.</li>
        <li><strong>Action:</strong> You follow the recommendation to set up geo-replication, so your application is more resilient to outages.</li>
    </ul>
    <li><strong>Security</strong></li>
    <p><strong>Purpose:</strong> Detects potential security issues and vulnerabilities to protect your resources from threats.</p>
    <p><strong>Example:</strong></p>
    <ul>
        <li><strong>Scenario:</strong> Azure Advisor notices that your virtual machines are not using encryption for their disks. This could expose your data if someone gains unauthorized access.</li>
        <li><strong>Action:</strong> The recommendation might be to enable disk encryption on your VMs. You implement encryption to safeguard your data.</li>
    </ul>
    <li><strong>Performance</strong></li>
    <p><strong>Purpose:</strong> Improves the speed and efficiency of your applications and resources.</p>
    <p><strong>Example:</strong></p>
    <ul>
        <li><strong>Scenario:</strong> Your web application is experiencing slow performance. Azure Advisor identifies that your application is using a basic-tier database that might be underpowered for your needs.</li>
        <li><strong>Action:</strong> It recommends upgrading to a higher performance database tier. After the upgrade, your application's performance improves, leading to a better user experience.</li>
    </ul>
    <li><strong>Operational Excellence</strong></li>
    <p><strong>Purpose:</strong> Helps you optimize your processes and manage resources effectively.</p>
    <p><strong>Example:</strong></p>
    <ul>
        <li><strong>Scenario:</strong> You have multiple resource groups in your Azure subscription, but they are not organized in a way that reflects your team structure.</li>
        <li><strong>Action:</strong> Azure Advisor suggests reorganizing your resources into different resource groups to streamline management and improve resource allocation. You follow this advice to enhance operational efficiency.</li>
    </ul>
    <li><strong>Cost</strong></li>
    <p><strong>Purpose:</strong> Reduces your overall spending on Azure by identifying areas where you can save money.</p>
    <p><strong>Example:</strong></p>
    <ul>
        <li><strong>Scenario:</strong> You have several underutilized VMs that are running but not being fully used.</li>
        <li><strong>Action:</strong> Azure Advisor recommends resizing or shutting down these VMs during off-peak hours to cut costs. You implement these changes and notice a reduction in your monthly Azure bill.</li>
    </ul>
</ol>

<p><strong>Using Azure Advisor</strong></p>
<ul>
    <li><strong>Accessing Recommendations:</strong> You can view Azure Advisor's recommendations in the Azure portal. It shows a dashboard with personalized advice based on your Azure environment.</li>
    <li><strong>Filtering:</strong> You can filter recommendations by specific subscriptions, resource groups, or services to focus on particular areas.</li>
    <li><strong>Notifications:</strong> Set up notifications to get alerts about new recommendations and stay updated on potential optimizations.</li>
</ul>
<img src="https://learn.microsoft.com/en-gb/training/wwl-azure/describe-monitoring-tools-azure/media/azure-advisor-dashboard-baca22e2.png">
<p><strong>Summary</strong></p>
<p>Azure Advisor helps you optimize your Azure setup by providing actionable recommendations. By following these suggestions, you can improve reliability, enhance security, boost performance, achieve operational excellence, and reduce costs. Whether you need to make your applications more resilient, protect your data, or cut expenses, Azure Advisor offers valuable guidance to help you manage your Azure resources more effectively.</p>

<p><strong>Azure Service Health</strong> helps you monitor and understand the health of your Azure resources and services. It combines three different tools to give you a complete view of your Azure environment. Here’s a breakdown of how it works and what each component does:</p>

<ol>
    <li><strong>Azure Status</strong>
        <p><strong>Purpose:</strong> Provides a broad view of Azure's global infrastructure health.</p>
        <p><strong>What It Shows:</strong> This tool displays the overall status of Azure services across all regions worldwide. It helps you see if there are any widespread outages or issues affecting multiple regions.</p>
        <p><strong>Example:</strong></p>
        <ul>
            <li><strong>Scenario:</strong> Imagine that Azure is experiencing a global outage affecting all customers, such as a major issue with Azure Storage services.</li>
            <li><strong>Action:</strong> You can check the Azure Status page to see a global view of this outage and understand its impact.</li>
        </ul>
    </li>
    <li><strong>Service Health</strong>
        <p><strong>Purpose:</strong> Gives a more focused view of the Azure services and regions you are specifically using.</p>
        <p><strong>What It Shows:</strong> This tool provides detailed information about the health of the Azure services and regions you have deployed resources in. It includes notifications about outages, planned maintenance, and other service-impacting events that might affect your resources.</p>
        <p><strong>Example:</strong></p>
        <ul>
            <li><strong>Scenario:</strong> You have virtual machines running in the East US region. Azure is planning maintenance in that region.</li>
            <li><strong>Action:</strong> Service Health will alert you about the planned maintenance and provide details on how it might affect your VMs. You can also set up alerts to notify you whenever there are changes or issues affecting the services you use.</li>
        </ul>
    </li>
    <li><strong>Resource Health</strong>
        <p><strong>Purpose:</strong> Offers a detailed view of the health of your specific Azure resources.</p>
        <p><strong>What It Shows:</strong> This tool focuses on the health of your individual Azure resources, such as virtual machines, databases, or storage accounts. It helps you monitor their availability and performance and provides alerts if there are issues.</p>
        <p><strong>Example:</strong></p>
        <ul>
            <li><strong>Scenario:</strong> One of your virtual machines is experiencing connectivity issues.</li>
            <li><strong>Action:</strong> Resource Health will show you the status of that specific VM and help you diagnose the issue. You can configure alerts to notify you of any availability changes or problems with your resources.</li>
        </ul>
    </li>
</ol>


<h1>Azure Health Service</h1>

<p><strong>Using Azure Service Health</strong></p>
<ol>
    <li><strong>Azure Status:</strong>
        <ul>
            <li><strong>Purpose:</strong> Understand global Azure service health.</li>
            <li><strong>How to Use:</strong> Visit the Azure Status page to check for global service outages or disruptions.</li>
        </ul>
    </li>
    <li><strong>Service Health:</strong>
        <ul>
            <li><strong>Purpose:</strong> Monitor the health of services and regions you use.</li>
            <li><strong>How to Use:</strong> Access the Service Health dashboard through the Azure portal. Set up alerts to receive notifications about issues affecting your services or planned maintenance.</li>
        </ul>
    </li>
    <li><strong>Resource Health:</strong>
        <ul>
            <li><strong>Purpose:</strong> Track the health of your specific resources.</li>
            <li><strong>How to Use:</strong> Check the Resource Health section for detailed information on the health of individual resources. Configure alerts through Azure Monitor for proactive issue management.</li>
        </ul>
    </li>
</ol>

<p><strong>Historical Alerts</strong></p>
<ul>
    <li><strong>Purpose:</strong> Review past incidents to understand trends and recurring issues.</li>
    <li><strong>How to Use:</strong> Access historical alerts in Azure Service Health to investigate past problems and see if they have become patterns that need addressing.</li>
</ul>

<p><strong>Support and Assistance</strong></p>
<ul>
    <li><strong>Purpose:</strong> Get help if your workload is affected.</li>
    <li><strong>How to Use:</strong> If an incident impacts your workload, Azure Service Health provides links to support resources where you can get assistance and further guidance.</li>
</ul>



<h1>Azure Moniter</h1>
<p><strong>Overview of Azure Monitor</strong></p>
<p>Azure Monitor is a comprehensive platform that helps you track the performance and health of your applications and resources. It collects data from various sources, analyzes it, and provides tools to visualize and act on that information. Azure Monitor supports monitoring of:</p>
<ul>
    <li>Azure resources</li>
    <li>On-premises resources</li>
    <li>Multi-cloud resources (e.g., virtual machines from other cloud providers)</li>
</ul>
<img src="https://learn.microsoft.com/en-gb/training/wwl-azure/describe-monitoring-tools-azure/media/azure-monitor-overview-614cd2fd.svg">
<p><strong>How Azure Monitor Works</strong></p>
<ol>
    <li><strong>Data Collection</strong>:
        <ul>
            <li><strong>Sources</strong>: Azure Monitor collects data from multiple layers of your architecture, including applications, operating systems, and network components.</li>
            <li><strong>Example</strong>: If you have a web application running on Azure, Azure Monitor can gather data on how the application is performing, how the underlying server is operating, and the network traffic.</li>
        </ul>
    </li>
    <li><strong>Data Storage</strong>:
        <ul>
            <li>The collected data is stored in central repositories for further analysis.</li>
            <li><strong>Example</strong>: Metrics like CPU usage and request response times are stored in a centralized place where they can be accessed for reporting and analysis.</li>
        </ul>
    </li>
    <li><strong>Data Visualization and Analysis</strong>:
        <ul>
            <li><strong>Real-time and Historical Views</strong>: You can see performance data in real time or look at historical data to identify trends.</li>
            <li><strong>Custom Views</strong>: Use tools like Power BI and Kusto queries to create specific reports and dashboards.</li>
            <li><strong>Example</strong>: You might set up a dashboard to show CPU usage trends over the past month to understand peak usage times.</li>
        </ul>
    </li>
    <li><strong>Alerts and Actions</strong>:
        <ul>
            <li><strong>Alerts</strong>: Azure Monitor can notify you when certain conditions are met, such as high CPU usage or errors.</li>
            <li><strong>Actionable Responses</strong>: You can configure it to trigger automatic actions like scaling resources up or down.</li>
            <li><strong>Example</strong>: If CPU usage on a virtual machine exceeds 80%, Azure Monitor can send you an email alert and automatically scale up the number of VM instances.</li>
        </ul>
    </li>
</ol>

<p><strong>Key Components of Azure Monitor</strong></p>
<ol>
    <li><strong>Azure Log Analytics</strong>:
        <ul>
            <li><strong>Purpose</strong>: Write and run queries on log data collected by Azure Monitor.</li>
            <li><strong>Simple Queries</strong>: Retrieve specific records and analyze them.</li>
            <li><strong>Advanced Queries</strong>: Perform complex data analysis and visualize results.</li>
            <li><strong>Example</strong>: You might write a query to find out how many times a specific error occurred in your application logs over the past week.</li>
        </ul>
    </li>
    <li><strong>Azure Monitor Alerts</strong>:
        <ul>
            <li><strong>Purpose</strong>: Automatically notify you when certain conditions or thresholds are met.</li>
            <li><strong>Types of Alerts</strong>: 
                <ul>
                    <li><strong>Metric-Based Alerts</strong>: Triggered based on numeric values like CPU usage.</li>
                    <li><strong>Log-Based Alerts</strong>: Triggered based on log events with complex conditions.</li>
                </ul>
            </li>
            <li><strong>Example</strong>: Set an alert to notify you when the response time of your web application exceeds 2 seconds.</li>
            <li><strong>Action Groups</strong>: Configure who gets notified and what actions to take.</li>
            <li><strong>Example</strong>: Set up an action group to send SMS notifications to the operations team and restart a service when a critical alert is triggered.</li>
        </ul>
    </li>
    <li><strong>Application Insights</strong>:
        <ul>
            <li><strong>Purpose</strong>: Monitor the performance and health of web applications.</li>
            <li><strong>Configuration</strong>: Install an SDK or use the Application Insights agent.</li>
            <li><strong>Key Metrics</strong>: 
                <ul>
                    <li>Request Rates</li>
                    <li>Response Times</li>
                    <li>Failure Rates</li>
                    <li>Dependency Rates</li>
                    <li>Page Views</li>
                </ul>
            </li>
            <li><strong>Example</strong>: Track the number of page views and response times for your e-commerce website to ensure it handles high traffic during sales events.</li>
            <li><strong>Synthetic Monitoring</strong>: Periodically send fake requests to check the application's status even during low activity.</li>
            <li><strong>Example</strong>: Use synthetic requests to test if your login page is working correctly every hour.</li>
        </ul>
    </li>
</ol>

<p><strong>Summary</strong></p>
<p>Azure Monitor provides a powerful way to keep an eye on your entire IT ecosystem, helping you ensure everything is running smoothly, quickly respond to issues, and make informed decisions based on comprehensive data analysis.</p>
