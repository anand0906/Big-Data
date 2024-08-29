<h1>Cost Effecting Factors</h1>
<p><strong>Understanding the factors that affect costs in Azure is crucial to effectively manage your expenses.</strong> Let's break down these factors in a simple and organized way, using real-time examples to make it easier to understand.</p>

<ol>
    <li>
        <p><strong>Resource Type</strong></p>
        <ul>
            <li><strong>Explanation:</strong> Different types of Azure resources (like virtual machines, storage accounts, databases, etc.) have different costs. The settings and configurations of these resources, as well as the region where they are deployed, also impact the price.</li>
            <li><strong>Real-Time Example:</strong> Imagine you want to store some files in Azure. You can choose a storage account with different types (e.g., Blob storage) and settings like performance tier (Standard or Premium), access tier (Hot or Cool), and redundancy options (how many copies of your data are kept and where). Each of these choices affects the cost. For instance, storing data in a Hot access tier (frequent access) is more expensive than in a Cool access tier (infrequent access).</li>
        </ul>
    </li>
    <li>
        <p><strong>Consumption</strong></p>
        <ul>
            <li><strong>Explanation:</strong> Azure operates on a pay-as-you-go model, meaning you pay for what you use. If you use more resources, you pay more, and if you use less, you pay less. Azure also offers discounts if you commit to using a certain amount of resources in advance.</li>
            <li><strong>Real-Time Example:</strong> Consider running a website on an Azure virtual machine (VM). If your website gets a lot of traffic this month, using more compute power, your bill will be higher. But if traffic is low next month, your costs will decrease. Additionally, if you know your website will consistently need a certain amount of compute power, you can reserve that capacity in advance and get a discount, reducing your overall cost.</li>
        </ul>
    </li>
    <li>
        <p><strong>Maintenance</strong></p>
        <ul>
            <li><strong>Explanation:</strong> Properly maintaining your Azure resources is important to avoid unnecessary costs. Sometimes, when you delete a resource, related resources (like storage or network configurations) may not be deleted automatically, which can lead to extra charges.</li>
            <li><strong>Real-Time Example:</strong> If you decommission a VM but forget to delete the associated storage, you're still paying for that storage. Regularly reviewing and cleaning up unused resources can save money.</li>
        </ul>
    </li>
    <li>
        <p><strong>Geography</strong></p>
        <ul>
            <li><strong>Explanation:</strong> The cost of Azure resources varies by region. This is due to differences in power costs, labor, taxes, and other factors in different parts of the world.</li>
            <li><strong>Real-Time Example:</strong> Deploying a VM in the United States might cost less than deploying the same VM in Japan because of the regional cost differences. Additionally, if your customers are primarily in Europe, it might be cheaper and faster to deploy resources in a European region to reduce data transfer costs.</li>
        </ul>
    </li>
    <li>
        <p><strong>Network Traffic</strong></p>
        <ul>
            <li><strong>Explanation:</strong> Data transfer within Azure regions (inbound traffic) is often free, but transferring data out of Azure (outbound traffic) incurs costs. These costs can vary depending on where the data is being sent.</li>
            <li><strong>Real-Time Example:</strong> If your application sends data from an Azure datacenter in the U.S. to a user in Asia, it will be more expensive than sending the same data to a user in the U.S. This is because of the higher cost of moving data across continents.</li>
        </ul>
    </li>
    <li>
        <p><strong>Subscription Type</strong></p>
        <ul>
            <li><strong>Explanation:</strong> Different Azure subscription types offer different pricing structures and benefits. Some subscriptions include free usage allowances, which can reduce your costs.</li>
            <li><strong>Real-Time Example:</strong> If you sign up for an Azure free trial, you get a certain amount of free usage for various Azure products for the first 12 months. If you exceed these limits or continue after the trial, you start paying according to Azure's standard rates.</li>
        </ul>
    </li>
    <li>
        <p><strong>Azure Marketplace</strong></p>
        <ul>
            <li><strong>Explanation:</strong> The Azure Marketplace offers various third-party solutions that integrate with Azure. While these solutions can add functionality, they also come with their own costs, which are set by the vendors.</li>
            <li><strong>Real-Time Example:</strong> Suppose you purchase a third-party firewall solution from the Azure Marketplace. You’ll pay for the Azure resources it uses (like VMs and storage) as well as the firewall software itself. These combined costs could be higher than just using Azure's native solutions.</li>
        </ul>
    </li>
</ol>

<p><strong>Summary:</strong> Managing costs in Azure involves understanding how different factors like resource type, consumption, maintenance, geography, network traffic, subscription type, and Azure Marketplace purchases affect your bill. By carefully choosing and maintaining your resources, you can optimize your Azure spending and avoid unnecessary charges.</p>


<h1>Savings</h1>
<p>Here’s a detailed explanation of factors that can help reduce costs in Azure, focusing on <strong>Reserved Instances</strong>, <strong>Reserved Capacity</strong>, <strong>Hybrid Use Benefit</strong>, and <strong>Spot Pricing</strong>.</p>

<ul>
    <li><strong>Azure Reservations</strong>
        <p><strong>Explanation:</strong> Azure Reservations allow you to purchase Azure services in advance for a duration of one or three years, which provides significant discounts compared to pay-as-you-go pricing. Reservations are available for various services and can drastically reduce costs for predictable, consistent workloads.</p>
        <p><strong>Types of Reservations:</strong></p>
        <ul>
            <li><strong>Reserved Instances (Azure Virtual Machines)</strong>: You can reserve virtual machine instances, locking in lower prices for one or three years.</li>
            <li><strong>Reserved Capacity</strong>: Available for services like Azure Storage, SQL Database vCores, Databricks DBUs (Databricks Units), and Cosmos DB RUs (Request Units).</li>
            <li><strong>Software Plans</strong>: You can reserve software licenses for products like Red Hat, Red Hat OpenShift, SUSE Linux, etc.</li>
        </ul>
        <p><strong>Real-Time Example:</strong> If you run a production environment with virtual machines that have consistent usage patterns, you can reserve those instances for three years, leading to significant cost savings compared to paying for the same VMs on a pay-as-you-go basis.</p>
    </li>
    <li><strong>Azure Spot VMs</strong>
        <p><strong>Explanation:</strong> Azure Spot VMs allow you to purchase unused Azure VM capacity at a significant discount. However, since this capacity is not guaranteed, it can be reclaimed by Azure at any time, making Spot VMs suitable for non-critical, interruptible workloads.</p>
        <p><strong>How It Works:</strong></p>
        <ul>
            <li><strong>Significant Discount:</strong> You get Azure VMs at a fraction of the usual cost.</li>
            <li><strong>Capacity Can Be Taken Away:</strong> Since Spot VMs utilize surplus capacity, Azure can reclaim the resources, so the VMs might be stopped or evicted with little notice.</li>
            <li><strong>Set Maximum Price:</strong> You can specify the maximum price you’re willing to pay for the Spot VMs, and if the current spot price exceeds this, your VMs will be evicted.</li>
        </ul>
        <p><strong>Best For:</strong> Batch processing, development/testing environments, large compute workloads, and other non-critical tasks where interruptions are acceptable.</p>
        <p><strong>Real-Time Example:</strong> If you are running a large-scale data processing job that can tolerate interruptions, using Spot VMs can significantly lower your costs. If Azure needs the capacity back, your job will be paused or stopped, but you save on costs when the VM is running.</p>
    </li>
    <li><strong>Hybrid Use Benefit</strong>
        <p><strong>Explanation:</strong> The Azure Hybrid Use Benefit allows you to bring your existing on-premises licenses (such as Windows Server or SQL Server) to Azure, thereby reducing the cost of running these services in the cloud. This is especially beneficial for organizations that have already invested in Microsoft software licenses.</p>
        <p><strong>Applicable To:</strong></p>
        <ul>
            <li><strong>Windows Server:</strong> Use your existing Windows Server licenses for Azure VMs.</li>
            <li><strong>RedHat and SUSE Linux:</strong> Apply existing licenses to Azure VMs running RedHat or SUSE Linux.</li>
            <li><strong>SQL Server:</strong> Utilize your SQL Server licenses for Azure SQL Database, Azure SQL Managed Instance, Azure SQL Server on VMs, and Azure Data Factory SQL Server Integration Services.</li>
        </ul>
        <p><strong>Real-Time Example:</strong> If your company has existing Windows Server licenses with Software Assurance, you can use these licenses on Azure VMs, reducing your overall cloud costs. Similarly, if you have SQL Server licenses, you can apply them to Azure SQL services, resulting in significant savings.</p>
    </li>
</ul>

<p><strong>Summary</strong></p>
<ul>
    <li><strong>Azure Reservations</strong> (Reserved Instances and Reserved Capacity) allow you to commit to using Azure services for 1 or 3 years at a lower price, ideal for stable and predictable workloads.</li>
    <li><strong>Azure Spot VMs</strong> offer a way to use unused Azure VM capacity at a reduced cost, perfect for workloads that can handle interruptions.</li>
    <li><strong>Hybrid Use Benefit</strong> lets you use your existing on-premises licenses in Azure, cutting down on licensing costs in the cloud.</li>
</ul>

<p>By utilizing these options, you can effectively manage and reduce your costs while using Azure services.</p>


<h1>Pricing Calculator and TCO Calculator</h1>
<p>The <strong>Pricing calculator</strong> and the <strong>Total Cost of Ownership (TCO) calculator</strong> are two tools that help you understand potential Azure expenses. Both calculators are accessible online, and both allow you to build out a configuration. However, the two calculators serve very different purposes.</p>

<h2>Pricing calculator</h2>
<p>The Pricing calculator is designed to provide you with an estimated cost for provisioning resources in Azure. You can get an estimate for individual resources, build out a solution, or use an example scenario to see an estimate of the Azure spend. The focus of the Pricing calculator is on the cost of provisioned resources in Azure.</p>

<p><strong>Note:</strong> The Pricing calculator is for informational purposes only. The prices are only an estimate. Nothing is provisioned when you add resources to the Pricing calculator, and you won't be charged for any services you select.</p>

<p>With the Pricing calculator, you can estimate the cost of any provisioned resources, including compute, storage, and associated network costs. You can even account for different storage options like storage type, access tier, and redundancy.</p>

<p><em>Screenshot of the Pricing calculator for reference.</em></p>

<h2>TCO calculator</h2>
<p>The TCO calculator is designed to help you compare the costs of running an on-premises infrastructure compared to an Azure Cloud infrastructure. With the TCO calculator, you enter your current infrastructure configuration, including servers, databases, storage, and outbound network traffic. The TCO calculator then compares the anticipated costs for your current environment with an Azure environment supporting the same infrastructure requirements.</p>

<p>With the TCO calculator, you enter your configuration, add in assumptions like power and IT labor costs, and are presented with an estimation of the cost difference to run the same environment in your current datacenter or in Azure.</p>

<h3>Define your requirements</h3>
<p>Before you run the Pricing calculator, you need to know what Azure services you need.</p>

<p>For example, for a basic web application hosted in your datacenter, you might run a configuration similar to the following:</p>
<ul>
    <li>An ASP.NET web application that runs on Windows. The web application provides information about product inventory and pricing.</li>
    <li>There are two virtual machines that are connected through a central load balancer.</li>
    <li>The web application connects to a SQL Server database that holds inventory and pricing information.</li>
</ul>

<p>To migrate to Azure, you might:</p>
<ul>
    <li>Use Azure Virtual Machines instances, similar to the virtual machines used in your datacenter.</li>
    <li>Use Azure Application Gateway for load balancing.</li>
    <li>Use Azure SQL Database to hold inventory and pricing information.</li>
</ul>

<p>Here's a diagram that shows the basic configuration:</p>
<p><em>A diagram showing a potential Azure solution for hosting an application.</em></p>

<p>In practice, you would define your requirements in greater detail. But here are some basic facts and requirements to get you started:</p>
<ul>
    <li>The application is used internally. It's not accessible to customers.</li>
    <li>This application doesn't require a massive amount of computing power.</li>
    <li>The virtual machines and the database run all the time (730 hours per month).</li>
    <li>The network processes about 1 TB of data per month.</li>
    <li>The database doesn't need to be configured for high-performance workloads and requires no more than 32 GB of storage.</li>
</ul>

<h3>Explore the Pricing calculator</h3>
<p>Let's start with a quick tour of the Pricing calculator.</p>
<p><a href="https://azure.microsoft.com/en-us/pricing/calculator/">Go to the Pricing calculator.</a></p>

<p>Notice the following tabs:</p>
<p><em>A screenshot of the Pricing calculator menu bar with the Products tab selected.</em></p>
<ul>
    <li><strong>Products</strong>: This is where you choose the Azure services that you want to include in your estimate. You'll likely spend most of your time here.</li>
    <li><strong>Example scenarios</strong>: Here you'll find several reference architectures, or common cloud-based solutions that you can use as a starting point.</li>
    <li><strong>Saved estimates</strong>: Here you'll find your previously saved estimates.</li>
    <li><strong>FAQs</strong>: Here you'll discover answers to frequently asked questions about the Pricing calculator.</li>
</ul>

<h3>Estimate your solution</h3>
<p>Here you add each Azure service that you need to the calculator. Then you configure each service to fit your needs.</p>

<p><strong>Tip:</strong> Make sure you have a clean calculator with nothing listed in the estimate. You can reset the estimate by selecting the trash can icon next to each item.</p>

<h4>Add services to the estimate</h4>
<p>On the <strong>Products</strong> tab, select the service from each of these categories:</p>
<table>
    <tr>
        <th>Category</th>
        <th>Service</th>
    </tr>
    <tr>
        <td>Compute</td>
        <td>Virtual Machines</td>
    </tr>
    <tr>
        <td>Databases</td>
        <td>Azure SQL Database</td>
    </tr>
    <tr>
        <td>Networking</td>
        <td>Application Gateway</td>
    </tr>
</table>

<p>Scroll to the bottom of the page. Each service is listed with its default configuration.</p>

<h4>Configure services to match your requirements</h4>
<p>Under <strong>Virtual Machines</strong>, set these values:</p>
<table>
    <tr>
        <th>Setting</th>
        <th>Value</th>
    </tr>
    <tr>
        <td>Region</td>
        <td>West US</td>
    </tr>
    <tr>
        <td>Operating system</td>
        <td>Windows</td>
    </tr>
    <tr>
        <td>Type</td>
        <td>(OS Only)</td>
    </tr>
    <tr>
        <td>Tier</td>
        <td>Standard</td>
    </tr>
    <tr>
        <td>Instance</td>
        <td>D2 v3</td>
    </tr>
    <tr>
        <td>Virtual machines</td>
        <td>2 x 730 Hours</td>
    </tr>
</table>

<p>Leave the remaining settings at their current values.</p>

<p>Under <strong>Azure SQL Database</strong>, set these values:</p>
<table>
    <tr>
        <th>Setting</th>
        <th>Value</th>
    </tr>
    <tr>
        <td>Region</td>
        <td>West US</td>
    </tr>
    <tr>
        <td>Type</td>
        <td>Single Database</td>
    </tr>
    <tr>
        <td>Backup storage tier</td>
        <td>RA-GRS</td>
    </tr>
    <tr>
        <td>Purchase model</td>
        <td>vCore</td>
    </tr>
    <tr>
        <td>Service tier</td>
        <td>General Purpose</td>
    </tr>
    <tr>
        <td>Compute tier</td>
        <td>Provisioned</td>
    </tr>
    <tr>
        <td>Generation</td>
        <td>Gen 5</td>
    </tr>
    <tr>
        <td>Instance</td>
        <td>8 vCore</td>
    </tr>
</table>

<p>Leave the remaining settings at their current values.</p>

<p>Under <strong>Application Gateway</strong>, set these values:</p>
<table>
    <tr>
        <th>Setting</th>
        <th>Value</th>
    </tr>
    <tr>
        <td>Region</td>
        <td>West US</td>
    </tr>
    <tr>
        <td>Tier</td>
        <td>Web Application Firewall</td>
    </tr>
    <tr>
        <td>Size</td>
        <td>Medium</td>
    </tr>
    <tr>
        <td>Gateway hours</td>
        <td>2 x 730 Hours</td>
    </tr>
    <tr>
        <td>Data processed</td>
        <td>1 TB</td>
    </tr>
    <tr>
        <td>Outbound data transfer</td>
        <td>5 GB</td>
    </tr>
</table>

<p>Leave the remaining settings at their current values.</p>

<h4>Review, share, and save your estimate</h4>
<p>At the bottom of the page, you see the total estimated cost of running the solution. You can change the currency type if you want.</p>

<p>At this point, you have a few options:</p>
<ul>
    <li>Select <strong>Export</strong> to save your estimate as an Excel document.</li>
    <li>Select <strong>Save</strong> or <strong>Save as</strong> to save your estimate to the Saved Estimates tab for later.</li>
    <li>Select <strong>Share</strong> to generate a link that you can send to others, which opens the estimate you just created.</li>
</ul>


<h1>Cost Management</h1>
<h2>Microsoft Cost Management</h2>

<p><strong>Microsoft Cost Management</strong> is a vital service within Azure that helps users manage and optimize their cloud expenses. As Azure is a global cloud provider, resources can be provisioned across different regions rapidly to meet various demands, such as increased traffic or testing new features. However, these rapid deployments can sometimes lead to unexpected costs, especially if resources are provisioned accidentally or forgotten. Cost Management helps you avoid such situations by providing tools to monitor, analyze, and control your Azure spending.</p>

<h3>Key Features of Microsoft Cost Management</h3>

<h4>1. Cost Management Overview</h4>
<ul>
  <li><strong>Cost Management</strong> allows users to quickly check their Azure resource costs, create alerts based on spending, and set up budgets that can automate the management of resources. This ensures that users have control over their expenses and can avoid unexpected charges.</li>
</ul>

<h4>2. Cost Analysis</h4>
<ul>
  <li><strong>Cost Analysis</strong> is a feature within Cost Management that provides a visual representation of Azure costs. It allows users to view their spending across different metrics, such as billing cycles, regions, and resources. This tool is essential for exploring and analyzing organizational costs, identifying spending trends, and estimating future expenses.</li>
  <li>Users can view aggregated costs by organization, helping them understand where costs are accrued and to identify spending trends over time. This feature is useful for estimating monthly, quarterly, or yearly cost trends against a set budget.</li>
</ul>

<h4>3. Cost Alerts</h4>
<ul>
  <li><strong>Cost Alerts</strong> offer a centralized location to monitor various types of alerts related to Azure spending. There are three primary types of alerts:</li>
  <ul>
    <li><strong>Budget Alerts</strong>: Notify users when spending reaches or exceeds the predefined budget threshold. These alerts can be based on either cost or usage.</li>
    <li><strong>Credit Alerts</strong>: Applicable for organizations with Enterprise Agreements (EAs), these alerts notify users when their Azure credit balance reaches 90% or 100%.</li>
    <li><strong>Department Spending Quota Alerts</strong>: Notify department owners when their spending reaches a specified percentage of the department’s quota.</li>
  </ul>
  <li>All alerts are visible in the Azure portal under cost alerts and are also sent via email to the designated recipients.</li>
</ul>

<h4>4. Budgets</h4>
<ul>
  <li><strong>Budgets</strong> allow users to set spending limits for Azure services. Budgets can be defined at various levels, such as by subscription, resource group, or service type. Once a budget is set, users can also configure budget alerts, which trigger notifications when spending reaches certain thresholds.</li>
  <li>Advanced budget features enable users to automate actions when budget conditions are met. For example, resources can be suspended or modified automatically when a budget threshold is reached, preventing overspending.</li>
</ul>

<h3>Summary</h3>
<p>Microsoft Cost Management provides essential tools to monitor, analyze, and control Azure spending. By leveraging cost analysis, cost alerts, and budgets, users can effectively manage their Azure resources and prevent unexpected charges, ensuring that cloud costs remain within their financial plans.</p>


<h1>Purpose Of Tags</h1>
<p>As cloud usage grows, staying organized becomes increasingly important. Tags are a valuable tool for managing and organizing cloud resources effectively. They provide additional metadata about your resources, which can be leveraged for various purposes:</p>

<h3>1. Resource Management</h3>
<ul>
  <li>Tags help locate and manage resources associated with specific workloads, environments, business units, and owners.</li>
</ul>

<h3>2. Cost Management and Optimization</h3>
<ul>
  <li>Tags enable you to group resources for cost reporting, internal cost allocation, budget tracking, and forecasting estimated costs.</li>
</ul>

<h3>3. Operations Management</h3>
<ul>
  <li>Tags allow you to group resources based on their criticality to business operations. This helps in formulating service-level agreements (SLAs) that guarantee uptime or performance.</li>
</ul>

<h3>4. Security</h3>
<ul>
  <li>Tags can classify data by its security level, such as public or confidential, enhancing data security management.</li>
</ul>

<h3>5. Governance and Regulatory Compliance</h3>
<ul>
  <li>Tags help identify resources that meet governance or regulatory compliance requirements, such as ISO 27001. They also support standards enforcement by requiring specific tags, such as owner or department names, on resources.</li>
</ul>

<h3>6. Workload Optimization and Automation</h3>
<ul>
  <li>Tags assist in visualizing resources involved in complex deployments. For example, tagging a resource with its associated workload or application name can facilitate automated tasks through software like Azure DevOps.</li>
</ul>

<h3>Managing Resource Tags</h3>
<p>Resource tags can be added, modified, or deleted using various methods, including Windows PowerShell, Azure CLI, Azure Resource Manager templates, REST API, or the Azure portal.</p>

<p>Azure Policy can enforce tagging rules and conventions, such as requiring specific tags on new resources or reapplying removed tags. Note that resources do not inherit tags from subscriptions or resource groups, allowing custom tagging schemas at different levels (resource, resource group, subscription, etc.).</p>

<h3>Example Tagging Structure</h3>
<p>A resource tag consists of a name and a value. Examples of common tags include:</p>
<ul>
  <li><strong>Name:</strong> The name of the application that the resource is part of.</li>
  <li><strong>CostCenter:</strong> The internal cost center code.</li>
  <li><strong>Owner:</strong> The name of the business owner responsible for the resource.</li>
  <li><strong>Environment:</strong> An environment name, such as "Prod," "Dev," or "Test."</li>
  <li><strong>Impact:</strong> The importance of the resource to business operations, such as "Mission-critical," "High-impact," or "Low-impact."</li>
</ul>

<p>It's not necessary to enforce a specific tag on all resources. For instance, only mission-critical resources may have the "Impact" tag, while non-tagged resources are considered non-critical.</p>


<h1>Microsoft Pureview</h1>
<p><strong>What is Microsoft Purview?</strong></p>
<p>Microsoft Purview is like a powerful tool that helps organizations keep track of all the data they have, no matter where it is stored. It ensures that this data is properly managed, protected, and used according to rules and regulations. Think of it as a central command center for data management.</p>

<p><strong>Key Features of Microsoft Purview</strong></p>

<ol>
  <li>
    <strong>Automated Data Discovery</strong>
    <p><strong>What it does:</strong> Microsoft Purview can automatically search through all the data your organization has, whether it's stored on your own servers, in cloud services like Azure, or even in other clouds like Amazon S3. It finds and catalogs this data so you know exactly what you have and where it is.</p>
    <p><strong>Example:</strong> Imagine you run a large company with data stored in different places—some in Azure, some in Google Cloud, and some on local servers. Without a tool like Purview, it would be hard to keep track of everything. Purview automatically finds all this data and organizes it, so you have a clear picture of your data landscape.</p>
  </li>
  
  <li>
    <strong>Sensitive Data Classification</strong>
    <p><strong>What it does:</strong> Purview can identify and label data that is sensitive, like personal information, credit card numbers, or health records. This helps ensure that sensitive data is protected and handled according to strict rules.</p>
    <p><strong>Example:</strong> Let’s say your company handles customer credit card information. Purview can automatically detect this type of data, label it as sensitive, and apply special protections to ensure it’s not mishandled or exposed to unauthorized users.</p>
  </li>
  
  <li>
    <strong>End-to-End Data Lineage</strong>
    <p><strong>What it does:</strong> Data lineage is like tracking the journey of data through your systems. Purview shows where the data comes from, how it’s processed, and where it ends up. This is important for understanding how data moves within your organization and ensuring it’s handled correctly.</p>
    <p><strong>Example:</strong> Suppose you have a report that shows sales figures. With Purview, you can trace where the data in that report came from—maybe it started in a sales database, was processed in an analytics tool, and then ended up in the report. This helps ensure the data is accurate and trustworthy.</p>
  </li>
</ol>

<p><strong>Two Main Areas of Microsoft Purview</strong></p>

<ol>
  <li>
    <strong>Risk and Compliance Solutions</strong>
    <p><strong>How it works:</strong> This part of Purview focuses on making sure your data is protected and that your organization complies with legal and regulatory requirements. It integrates with Microsoft 365 tools like Teams, OneDrive, and Exchange to monitor and protect data.</p>
    <p><strong>Example:</strong> Imagine your company needs to comply with data protection laws like GDPR. Purview can automatically monitor all communications and data exchanges within your organization, like emails and file sharing. If someone tries to send a sensitive document through email, Purview can block it or alert the sender that they’re violating company policies.</p>
  </li>
  
  <li>
    <strong>Unified Data Governance</strong>
    <p><strong>How it works:</strong> This part of Purview helps manage all your data, no matter where it’s stored. It allows you to create a complete map of your data, see how it’s classified, and track how it’s being used across different platforms.</p>
    <p><strong>Example:</strong> Suppose your organization has data stored in Azure, an SQL database, and Amazon S3. Purview lets you manage all this data from one place. You can set rules for who can access the data, ensure sensitive information is properly protected, and generate insights about how the data is being used.</p>
  </li>
</ol>

<p><strong>Real-Life Scenario</strong></p>
<p>Let’s say you work for a hospital that has patient records stored in different systems—some on local servers, some in the cloud, and some in third-party applications. These records include sensitive information like medical histories and personal details. Without a unified tool, it would be difficult to manage all this data and ensure it’s protected.</p>
<p>With Microsoft Purview:</p>

<ul>
  <li>You can automatically discover and classify all the patient records, regardless of where they are stored.</li>
  <li>Purview will label the sensitive information and ensure it’s protected according to healthcare regulations like HIPAA.</li>
  <li>You can track the entire journey of each patient record, from when it was created to where it’s stored and how it’s accessed.</li>
  <li>You can manage who has access to the data and ensure that only authorized personnel can view or edit the records.</li>
</ul>

<p>In summary, Microsoft Purview provides a comprehensive solution for managing, protecting, and governing data across an organization, ensuring that sensitive information is handled properly and that the organization stays compliant with relevant laws and regulations.</p>


<h1>Azure Policy</h1>
<p><strong>Azure Policy</strong> is a service in Microsoft Azure that helps organizations ensure their resources stay compliant with corporate standards by allowing them to create, assign, and manage policies that govern resource configurations.</p>

<p><strong>Key Features of Azure Policy</strong></p>

<ol>
  <li>
    <strong>Creating and Managing Policies:</strong>
    <p>Azure Policy allows you to define rules, known as policies, that enforce specific configurations on your resources. These policies help ensure that all resources in your Azure environment meet your organization's standards.</p>
    <p><strong>Example:</strong> Suppose your organization requires all virtual machines (VMs) to be encrypted for security reasons. You can create an Azure Policy that ensures every VM created in your environment is encrypted. If someone tries to create a VM without encryption, Azure Policy can block it or notify you.</p>
  </li>

  <li>
    <strong>Policy Definition:</strong>
    <p>A policy in Azure defines a condition and an effect. The condition specifies what should be checked, and the effect determines what happens if the condition is met.</p>
    <p><strong>Example:</strong> You can define a policy with a condition that checks if a resource is located in a specific region (like "East US"). The effect could be to deny the creation of resources outside that region. This ensures that all resources are deployed only in approved locations.</p>
  </li>

  <li>
    <strong>Policy Initiatives:</strong>
    <p>An initiative is a group of related policies that work together to achieve a larger goal. Initiatives help manage multiple policies as a single unit.</p>
    <p><strong>Example:</strong> Suppose your organization wants to enhance security across all its resources. You could use the "Enable Monitoring in Azure Security Center" initiative, which includes multiple policies that monitor things like unencrypted databases, OS vulnerabilities, and missing endpoint protection. By applying this initiative, you ensure comprehensive security monitoring across your environment.</p>
  </li>

  <li>
    <strong>Scope and Inheritance:</strong>
    <p>Policies can be applied at different levels, such as management groups, subscriptions, resource groups, or individual resources. When you set a policy at a higher level, it automatically applies to all resources under that level.</p>
    <p><strong>Example:</strong> If you apply a policy to a resource group, all resources within that group inherit the policy. For instance, if you set a policy that requires all storage accounts to be encrypted, every storage account within that resource group will automatically comply with this rule.</p>
  </li>

  <li>
    <strong>Built-In and Custom Policies:</strong>
    <p>Azure Policy provides built-in policies that cover common scenarios, like enforcing security standards or managing costs. However, you can also create custom policies to meet your organization’s specific needs.</p>
    <p><strong>Example:</strong> If you want to enforce a rule that only allows certain types of VMs to be created, you can either use a built-in policy or create a custom one tailored to your exact requirements.</p>
  </li>

  <li>
    <strong>Policy Enforcement and Remediation:</strong>
    <p>Azure Policy can prevent non-compliant resources from being created in the first place, and it can also audit existing resources to ensure they meet your policies. If a resource becomes non-compliant, Azure Policy can automatically fix the issue.</p>
    <p><strong>Example:</strong> Let’s say your policy requires that all resources in a resource group have a specific tag (like "Environment: Production"). If a resource is missing this tag, Azure Policy can automatically add it, ensuring consistent tagging across your resources.</p>
  </li>

  <li>
    <strong>Exclusions:</strong>
    <p>You can exclude specific resources or groups of resources from a policy if needed. This allows flexibility in managing exceptions.</p>
    <p><strong>Example:</strong> If you have a legacy system that cannot comply with a certain policy, you can exclude it from that policy while still enforcing the policy on all other resources.</p>
  </li>
</ol>

<p><strong>Real-Life Scenario</strong></p>
<p>Imagine you are managing IT for a financial services company that must comply with strict regulations. Your company uses Azure to host various applications and services. To ensure compliance and security, you might use Azure Policy as follows:</p>

<ul>
  <li><strong>Security Compliance:</strong> Create policies that enforce encryption on all storage accounts and virtual machines. If someone tries to create an unencrypted resource, Azure Policy blocks it or flags it for review.</li>
  <li><strong>Cost Management:</strong> Apply a policy that limits the types of VMs that can be created to control costs. For example, you can restrict the creation of only specific VM sizes that fit within your budget.</li>
  <li><strong>Tagging and Organization:</strong> Implement a policy that requires all resources to have specific tags, like "Department" and "Environment," to ensure resources are properly organized and accounted for.</li>
</ul>

<p><strong>Azure Policy with Azure DevOps</strong></p>
<p>Azure Policy also integrates with Azure DevOps, ensuring that policies are applied during the deployment process. This helps maintain compliance from the very start of your development pipeline.</p>

<p><strong>Example:</strong> If you have a policy that requires all deployments to specific regions, Azure DevOps can check this policy during the deployment process. If a deployment is targeted at an unauthorized region, the pipeline can halt, ensuring that only compliant deployments are executed.</p>

<p><strong>Summary</strong></p>
<p>Azure Policy is a powerful tool that helps organizations enforce compliance, security, and governance across their Azure resources. By defining and applying policies, you can ensure that all resources in your environment adhere to your organization’s standards, preventing issues before they occur and automatically remediating non-compliance when necessary.</p>


<h1>Resource Locks</h1>
<p><strong>Resource Lock:</strong> A resource lock prevents resources from being accidentally deleted or changed.</p>

<p>Even with Azure role-based access control (Azure RBAC) policies in place, there's still a risk that people with the right level of access could delete critical cloud resources. Resource locks prevent resources from being deleted or updated, depending on the type of lock. Resource locks can be applied to individual resources, resource groups, or even an entire subscription. Resource locks are inherited, meaning that if you place a resource lock on a resource group, all of the resources within the resource group will also have the resource lock applied.</p>

<p><strong>Types of Resource Locks:</strong></p>

<ul>
    <li><strong>Delete:</strong> Authorized users can still read and modify a resource, but they can't delete the resource.</li>
    <li><strong>ReadOnly:</strong> Authorized users can read a resource, but they can't delete or update the resource. Applying this lock is similar to restricting all authorized users to the permissions granted by the Reader role.</li>
</ul>

<p><strong>How do I manage resource locks?</strong></p>

<p>You can manage resource locks from the Azure portal, PowerShell, the Azure CLI, or from an Azure Resource Manager template.</p>

<p>To view, add, or delete locks in the Azure portal, go to the Settings section of any resource's Settings pane in the Azure portal.</p>

<p><strong>How do I delete or change a locked resource?</strong></p>

<p>Although locking helps prevent accidental changes, you can still make changes by following a two-step process.</p>

<p>To modify a locked resource, you must first remove the lock. After you remove the lock, you can apply any action you have permissions to perform. Resource locks apply regardless of RBAC permissions. Even if you're an owner of the resource, you must still remove the lock before you can perform the blocked activity.</p>

<p><strong>What is an Azure Resource Lock?</strong></p>

<ul>
    <li><strong>Designed to prevent accidental deletion and/or modification</strong></li>
    <li><strong>Used in conjunction with RBAC</strong></li>
    <li><strong>Two types of locks:</strong>
        <ul>
            <li><strong>Read-only (ReadOnly):</strong> Only read actions are allowed</li>
            <li><strong>Delete (CanNotDelete):</strong> All actions except delete are allowed</li>
        </ul>
    </li>
    <li><strong>Scopes are hierarchical (inherited):</strong>
        <ul>
            <li>Subscriptions > Resource Groups > Resources</li>
        </ul>
    </li>
    <li><strong>Management Groups can’t be locked</strong></li>
    <li><strong>Only Owner and User Access Administrator roles can manage locks (built-in roles)</strong></li>
</ul>

<p><strong>Exercise - Configure a resource lock</strong></p>

<p>In this exercise, you’ll create a resource and configure a resource lock. Storage accounts are one of the easiest types of resource locks to quickly see the impact, so you’ll use a storage account for this exercise.</p>

<p>This exercise is a Bring your own subscription exercise, meaning you’ll need to provide your own Azure subscription to complete the exercise. Don’t worry though, the entire exercise can be completed for free with the 12-month free services when you sign up for an Azure account.</p>

<p>For help with signing up for an Azure account, see the Create an Azure account learning module.</p>

<p>Once you’ve created your free account, follow the steps below. If you don’t have an Azure account, you can review the steps to see the process for adding a simple resource lock to a resource.</p>

<ol>
    <li><strong>Task 1: Create a resource</strong>
        <ul>
            <li>Sign in to the Azure portal at <a href="https://portal.azure.com">https://portal.azure.com</a></li>
            <li>Select Create a resource.</li>
            <li>Under Categories, select Storage.</li>
            <li>Under Storage Account, select Create.</li>
            <li>On the Basics tab of the Create storage account blade, fill in the following information. Leave the defaults for everything else.</li>
            <ul>
                <li><strong>Resource group:</strong> Create new</li>
                <li><strong>Storage account name:</strong> Enter a unique storage account name</li>
                <li><strong>Location:</strong> Default</li>
                <li><strong>Performance:</strong> Standard</li>
                <li><strong>Redundancy:</strong> Locally redundant storage (LRS)</li>
            </ul>
            <li>Select Review + Create to review your storage account settings and allow Azure to validate the configuration.</li>
            <li>Once validated, select Create. Wait for the notification that the account was successfully created.</li>
            <li>Select Go to resource.</li>
        </ul>
    </li>
    <li><strong>Task 2: Apply a read-only resource lock</strong>
    	<img src="https://learn.microsoft.com/en-gb/training/wwl-azure/describe-features-tools-azure-for-governance-compliance/media/read-only-lock-e7777623.png">
        <ul>
            <li>In this task, you apply a read-only resource lock to the storage account. What impact do you think that will have on the storage account?</li>
            <li>Scroll down until you find the Settings section of the blade on the left of the screen.</li>
            <li>Select Locks.</li>
            <li>Select + Add.</li>
            <li>Enter a Lock name.</li>
            <li>Verify the Lock type is set to Read-only.</li>
            <li>Select OK.</li>
        </ul>
    </li>
    <li><strong>Task 3: Add a container to the storage account</strong>
    	<img src="https://learn.microsoft.com/en-gb/training/wwl-azure/describe-features-tools-azure-for-governance-compliance/media/read-only-lock-e7777623.png">
        <ul>
            <li>In this task, you add a container to the storage account, this container is where you can store your blobs.</li>
            <li>Scroll up until you find the Data storage section of the blade on the left of the screen.</li>
            <li>Select Containers.</li>
            <li>Select + Container.</li>
            <li>Enter a container name and select Create.</li>
            <li>You should receive an error message: Failed to create storage container.</li>
            <li>The error message lets you know that you couldn't create a storage container because a lock is in place. The read-only lock prevents any create or update operations on the storage account, so you're unable to create a storage container.</li>
            <img src="https://learn.microsoft.com/en-gb/training/wwl-azure/describe-features-tools-azure-for-governance-compliance/media/failed-to-create-warning-291af699.png">
        </ul>
    </li>
    <li><strong>Task 4: Modify the resource lock and create a storage container</strong>
    	<img src="https://learn.microsoft.com/en-gb/training/wwl-azure/describe-features-tools-azure-for-governance-compliance/media/resource-lock-change-e5281189.png">
        <ul>
            <li>Scroll down until you find the Settings section of the blade on the left of the screen.</li>
            <li>Select Locks.</li>
            <li>Select the read-only resource lock you created.</li>
            <li>Change the Lock type to Delete and select OK.</li>
            <li>Scroll up until you find the Data storage section of the blade on the left of the screen.</li>
            <li>Select Containers.</li>
            <li>Select + Container.</li>
            <li>Enter a container name and select Create.</li>
            <li>Your storage container should appear in your list of containers.</li>
            <li>You can now understand how the read-only lock prevented you from adding a container to your storage account. Once the lock type was changed (you could have removed it instead), you were able to add a container.</li>
        </ul>
    </li>
    <li><strong>Task 5: Delete the storage account</strong>
    	<img src="https://learn.microsoft.com/en-gb/training/wwl-azure/describe-features-tools-azure-for-governance-compliance/media/storage-overview-page-ec75f9e6.png">
        <ul>
            <li>You'll actually do this last task twice. Remember that there is a delete lock on the storage account, so you won't actually be able to delete the storage account yet.</li>
            <li>Scroll up until you find Overview at the top of the blade on the left of the screen.</li>
            <li>Select Overview.</li>
            <li>Select Delete.</li>
            <li>You should get a notification letting you know you can't delete the resource because it has a delete lock. In order to delete the storage account, you'll need to remove the delete lock.</li>
        </ul>
    </li>
    <li><strong>Task 6: Remove the delete lock and delete the storage account</strong>
    	<img src="https://learn.microsoft.com/en-gb/training/wwl-azure/describe-features-tools-azure-for-governance-compliance/media/storage-delete-lock-warning-5ea6faa5.png">
        <ul>
            <li>In the final task, you remove the resource lock and delete the storage account from your Azure account. This step is important. You want to make sure you don't have any idle resource just sitting in your account.</li>
            <li>Select your storage account name in the breadcrumb at the top of the screen.</li>
            <li>Scroll down until you find the Settings section of the blade on the left of the screen.</li>
            <li>Select Locks.</li>
            <li>Select Delete.</li>
            <li>Select Home in the breadcrumb at the top of the screen.</li>
            <li>Select Storage accounts</li>
            <li>Select the storage account you used for this exercise.</li>
            <li>Select Delete.</li>
            <li>To prevent accidental deletion, Azure prompts you to enter the name of the storage account you want to delete. Enter the name of the storage account and select Delete.</li>
            <li>You should receive a message that the storage account was deleted. If you go Home > Storage accounts, you should see that the storage account you created for this exercise is gone.</li>
        </ul>
        <img src="https://learn.microsoft.com/en-gb/training/wwl-azure/describe-features-tools-azure-for-governance-compliance/media/storage-account-delete-f4d60c3b.png">
    </li>
</ol>

<p>Congratulations! You've completed configuring, updating, and removing a resource lock on an Azure resource.</p>

<p><strong>Important:</strong> Make sure you complete Task 6, the removal of the storage account. You are solely responsible for the resources in your Azure account. Make sure you clean up your account after completing this exercise.</p>


<h1>Microsoft Service Trust Portal</h1>
<p><strong>The Microsoft Service Trust Portal</strong> is a centralized platform designed to provide users with access to a wide range of information, tools, and resources about Microsoft's security, privacy, and compliance practices. It's especially useful for organizations that use Microsoft cloud services, as it helps them understand how Microsoft protects their data and ensures compliance with various regulations.</p>

<p><strong>Purpose of the Service Trust Portal</strong></p>

<p>The main purpose of the Service Trust Portal is to give organizations confidence in the security and privacy of Microsoft's cloud services. This is important because companies often handle sensitive data, such as customer information, financial records, or intellectual property, in the cloud. They need to ensure that this data is protected from unauthorized access, breaches, or other risks.</p>

<p><strong>Example:</strong> Imagine you're running a healthcare organization that uses Microsoft Azure to store patient records. These records contain highly sensitive information, and you need to be absolutely sure that this data is secure and complies with healthcare regulations, such as HIPAA in the United States. The Service Trust Portal would be your go-to resource to understand how Microsoft ensures that its cloud services meet these regulatory requirements and protect your patient data.</p>

<p><strong>Key Features of the Service Trust Portal</strong></p>

<ol>
    <li><strong>Access to Compliance Information:</strong>
        <ul>
            <li>The portal provides detailed information about how Microsoft implements controls and processes to protect customer data in the cloud.</li>
            <li>For example, if you need to know how Microsoft ensures that its services comply with GDPR (General Data Protection Regulation), you can find documents and resources on the Service Trust Portal that explain these processes.</li>
        </ul>
    </li>
    <li><strong>Authentication and Access:</strong>
        <ul>
            <li>Some resources on the portal require you to sign in with your Microsoft cloud services account, which is often linked to your organization's Microsoft Entra account.</li>
            <li>This is like having a special key to access sensitive documents, ensuring that only authorized users within your organization can see this information.</li>
        </ul>
    </li>
    <li><strong>My Library:</strong>
        <ul>
            <li>The portal allows you to save important documents to a personal library, making it easy to access them later.</li>
            <li>For example, if you're preparing for an audit and need to frequently reference certain compliance documents, you can pin them to your library for quick access.</li>
        </ul>
    </li>
    <li><strong>Document Updates and Notifications:</strong>
        <ul>
            <li>The portal can notify you when important documents are updated. This is crucial because regulations and compliance requirements can change, and you need to stay up-to-date.</li>
            <li>If there’s a change in GDPR guidelines, the portal will alert you when relevant documents are updated, so you can ensure your organization remains compliant.</li>
        </ul>
    </li>
</ol>

<p><strong>Real-Time Example:</strong> Suppose your organization is preparing for a security audit, and you need to demonstrate that your use of Microsoft cloud services meets specific regulatory standards. The Service Trust Portal would be your primary resource for gathering all the necessary documents, such as audit reports, compliance certificates, and security control details. By using this portal, you can efficiently collect and present the information needed to pass the audit and ensure that your organization continues to operate within the required legal and regulatory frameworks.</p>

<p>In summary, the Service Trust Portal is a critical resource for any organization that uses Microsoft cloud services, providing transparency and confidence in Microsoft's commitment to security, privacy, and compliance.</p>
