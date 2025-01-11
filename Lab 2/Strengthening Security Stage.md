

In the first stage of this lab, I established a foundational structure using Terraform to automate the setup of users, groups, and roles in Azure AD. This created a scalable, consistent hierarchy that reflects a real-world organization. Now, the focus shifts to enhancing the security of this environment.



![[cyber security.webp]]



This stage emphasizes proactive safeguards, ensuring access is both secure and tightly controlled. Advanced security settings, including Multi-Factor Authentication (MFA) and Just-In-Time (JIT) access, work together to prevent unauthorized access and reduce risks. Additionally, compliance policies and tools like Privileged Identity Management (PIM) ensure that permissions are granted only when necessary, aligning with the principle of least privilege.

#### What This Means

This stage demonstrates how security can complement usability. Sensitive information is protected, industry standards are met, and operational challenges are anticipated. In practice, this results in fewer breaches, simplified audits, and reduced administrative overhead.

These efforts showcase not only technical expertise but also the ability to design scalable, secure, and compliant systems. By addressing challenges such as phishing and excessive permissions, this lab ensures a resilient and reliable Azure AD environment.

![[Security framwork.png |700]]
### MFA Setup




![[mfa-visual.jpg | 750]]

I manually enabled Multi-Factor Authentication (MFA) for all current employees in the company using the free version of Azure AD. While I don't have access to premium licenses, enabling MFA is the next best step I can take to secure our environment and protect both the company and its users.

MFA adds an extra layer of security to user accounts by requiring not just a password, but a second method of verification, like a code sent to their phone or an app-based prompt. This means that even if someone manages to steal a password, they still can’t access the account without the second factor.

![[Screenshot 2024-12-11 122520.png | 500]]



This helps secure the company by:

- **Preventing unauthorized access**: It makes it much harder for attackers to break into accounts, even if passwords are compromised.
- **Protecting sensitive resources**: MFA ensures that only verified employees can access critical systems and data.
- **Defending against common threats**: It’s highly effective against phishing attacks and password guessing.

By implementing MFA for all employees, I’ve taken an essential step to strengthen the company’s security, even within the limits of a free Azure license. It’s a simple but powerful way to reduce risks and protect the organization.


![[Screenshot 2024-12-26 153226.png]]



Additionally, I set up a Conditional Access policy requiring all users to use MFA. This policy automates the enforcement of MFA for all users without needing to manually enable it for each account, ensuring a consistent and streamlined security approach.

![[Screenshot 2024-12-21 131924.png]]

I enforced Multi-Factor Authentication (MFA) for Rachel Platinum because she holds an executive-level position in the company. As an executive, Rachel has access to sensitive information and critical resources, making it essential to apply stronger security measures to her account. Enforcing MFA ensures that her account is protected by requiring an additional layer of authentication beyond just a password.

The **Enforce MFA** feature mandates that Rachel completes MFA registration during her next login and requires her to verify her identity with a second factor every time she signs in. This feature blocks access to her account until MFA is fully set up, ensuring no unauthorized access can occur. By enforcing MFA, Rachel’s account is more secure against threats like phishing and credential theft, safeguarding the company's most sensitive data.

![[Screenshot 2024-12-11 125333.png | 600]]





### MFA login process

Once Rachel login to the portal, it will ask for a password, then the system will walk Rachel to setup the Microsoft Authenticator.

![[Screenshot 2024-12-11 181909.png]]


![[Screenshot 2024-12-11 181933.png]]


![[Screenshot 2024-12-11 181959.png | 500]]


![[Screenshot 2024-12-11 182030.png | 600]]

![[Screenshot 2024-12-11 182053.png | 600]]

The rest of the process involves setting up Microsoft Authenticator to generate a time-based code within the app.


## Conditional Access

To take this further, I also set up an additional Conditional Access policy named "High Risk" to boost security. Conditional Access is a powerful tool in Azure AD that allows me to enforce policies based on specific conditions, such as user behavior, device status, and application usage. It helps me tailor security controls to match the risk levels of various situations, ensuring the organization stays protected while maintaining usability.


![[Screenshot 2024-12-23 134856 1.png | 800]]

![[Screenshot 2024-12-21 134937.png | 500]]


    
    
![[Screenshot 2024-12-21 135157.png | 200]]
Here’s how I configured it:

- **User Risk**: I set it to block access for users identified as high risk. High-risk users typically indicate a compromised account or suspicious activity, so this ensures that these accounts cannot access sensitive resources until the issues are resolved.
    
- **Sign-In Risk**: I configured it to restrict sign-ins for users flagged with medium to high-risk activity. This setting helps catch unusual login behavior, such as sign-ins from unfamiliar locations or devices, and blocks access until further verification can take place.
    
- **Client Apps**: The policy applies to browser and mobile app clients while excluding legacy authentication methods like Exchange ActiveSync and other outdated protocols. Legacy authentication methods are vulnerable to attacks because they lack modern security features. By disabling these, I reduce potential vulnerabilities significantly.
    
- **Grant Access with MFA**: I made sure that users granted access must satisfy the requirement for MFA. This ensures that even if some risks pass initial detection, MFA provides an extra barrier to protect accounts.

These choices strike a thoughtful balance between maintaining strong security and ensuring everyday usability. With Conditional Access, I’ve built policies that dynamically adjust to various risk levels, removing the need for constant manual intervention. This setup directly protects against threats like phishing, credential theft, and brute-force attacks, which are some of the most common challenges organizations face. At the same time, it keeps the environment user-friendly, allowing legitimate users to work without unnecessary roadblocks. This combination of automation and practicality is a core part of making the security measures both robust and seamless for everyone involved.



### Policy Setup


![[azure-policy.png | 500]]


Azure Policies are like rules that help companies keep their cloud resources safe, organized, and working properly. They make sure everything follows the company’s guidelines and security standards. With these policies, companies can set up automatic rules to keep their resources secure and well-organized without doing everything manually.

**Why Companies Should Use Azure Policies** Azure Policies help companies stay safe, follow the rules, and keep things running smoothly. For example, a company can require all their resources to have labels (called tags) to know which department owns them or make sure only secure web connections (HTTPS) are allowed. By using Azure Policies, companies can:


![[Pasted image 20241226220014.png]]


**Example** Let’s say a bank wants to make sure all their data is encrypted (scrambled so only authorized people can read it). Azure Policies can automatically enforce this rule, making it impossible to store unencrypted data. Another example is blocking the creation of publicly accessible servers to keep customer data safe.


## Common Policies 
These are some of the most common policies that I pick to use in my lab. They help this organizations secure its resources, and manage their cloud environment more effectively. 

### 1. **Block Public IP Addresses**

- **What It Does**: Stops resources like servers or databases from being assigned public internet addresses.
    
- **Why It's Important**: Public IPs make resources visible to hackers on the internet. This rule ensures only authorized users can access them through secure channels.
    

### 2. **Enforce Resource Tagging**

- **What It Does**: Makes sure all resources are labeled with tags like `Department` or `Project`.
    
- **Why It's Important**: Tags help keep resources organized and make it easier to track who owns what. This is super helpful for managing costs and audits.
    

### 3. **Restrict Resource Locations**

- **What It Does**: Limits where resources can be created, like only in regions like `East US`.
    
- **Why It's Important**: Ensures resources are in regions that comply with legal rules or are close to users for better speed.
    

### 4. **Ensure MFA Is Enabled for Administrators**

- **What It Does**: Flags admin accounts that don’t have Multi-Factor Authentication (MFA) enabled.
    
- **Why It's Important**: MFA adds an extra layer of protection to prevent unauthorized access to high-level accounts.
    

    


![[Screenshot 2024-12-11 132429.png | 600]]


### Create Deny Public IP
The Block Public IP policy I had to created in json as I could not find this listed. 


```
{
  "properties": {
    "displayName": "Deny Public IP Addresses for Resources",
    "policyType": "Custom",
    "mode": "Indexed",
    "description": "Blocks resources from having public IP addresses to improve security.",
    "metadata": {
      "version": "1.0.0",
      "category": "Security"
    },
    "parameters": {},
    "policyRule": {
      "if": {
        "allOf": [
          {
            "field": "type",
            "in": [
              "Microsoft.Network/publicIPAddresses",
              "Microsoft.Compute/virtualMachines",
              "Microsoft.Network/loadBalancers"
            ]
          },
          {
            "field": "Microsoft.Network/publicIPAddresses/ipAddress",
            "exists": true
          }
        ]
      },
      "then": {
        "effect": "Deny"
      }
    }
  }
}

```


### Enabled MFA for Admin
I had to create this policy in custom as this was not in the list as well.

```
{
  "properties": {
    "displayName": "Require MFA for Admin Accounts",
    "policyType": "Custom",
    "mode": "All",
    "description": "Ensures that multi-factor authentication (MFA) is enabled for all accounts with administrative privileges.",
    "metadata": {
      "version": "1.0.0",
      "category": "Identity and Access Management"
    },
    "parameters": {},
    "policyRule": {
      "if": {
        "allOf": [
          {
            "field": "type",
            "equals": "Microsoft.Authorization/roleAssignments"
          },
          {
            "field": "Microsoft.Authorization/roleAssignments/roleDefinitionId",
            "equals": "/providers/Microsoft.Authorization/roleDefinitions/8e3af657-a8ff-443c-a75c-2fe8c4bcb635"
          },
          {
            "field": "Microsoft.Authorization/roleAssignments/principalType",
            "equals": "User"
          }
        ]
      },
      "then": {
        "effect": "Deny"
      }
    }
  }
}


```

After all 4 policies were added my Policy portal show all the policies I enabled.

![[Screenshot 2024-12-11 171125.png | 900]]




## Strengthening Security with Advanced Policies and Protections


Building on the foundational security measures in place, I added a series of advanced configurations to address specific vulnerabilities and bolster the overall protection of my lab environment. These settings are designed to target common risks while enhancing usability and control for administrators. Here are the configurations I implemented:

- **Password Protection**: I set up custom banned passwords like "Password123" and "Welcome123" to prevent users from choosing weak and predictable passwords. I enabled the custom banned password list and enforced password protection for Windows Server Active Directory. These measures make it harder for attackers to exploit commonly used or default passwords.

	![[Screenshot 2024-12-17 164614.png|600]]

- **Lockout Policy**: To mitigate brute-force attacks, I configured the lockout threshold to trigger after 5 failed attempts and set the lockout duration to 300 seconds. This ensures that repeated failed login attempts will temporarily block access, reducing the risk of automated attacks.
    
- **Guest User Sharing Settings**: I disabled the option for users to add new guests to the organization. By doing this, only administrators can invite and manage guest accounts, which helps maintain control over who can access the organization’s resources and prevents potential misuse of guest access.

	![[Screenshot 2024-12-17 111736.png|500]]
- **Password Expiration Policy**: I set passwords to expire every 90 days. This encourages users to change their credentials regularly, reducing the risk of long-term password compromise. While Azure recommends passwords that never expire when using MFA, I opted for expiration to fit the specific goals of this lab.
    ![[Screenshot 2024-12-17 111234.png|500]]

- **Idle Session Timeout**: I configured the idle session timeout for Microsoft 365 web apps to automatically sign out users after 3 hours of inactivity. This reduces the risk of unauthorized access from unattended or forgotten sessions, ensuring sensitive data remains secure even in cases of negligence. They demonstrate a layered approach to securing an Azure AD environment, providing real-world relevance to this lab.
	![[Screenshot 2024-12-17 111448.png|500]]
---

In this stage, I focused on implementing foundational security measures to enhance the environment's protection and organization. Multi-Factor Authentication (MFA) was a critical step, ensuring that all employee accounts had an extra layer of security. For executive-level accounts, like Rachel Platinum’s, I enforced MFA to safeguard high-value resources. Additionally, I created and assigned key Azure Policies, including custom definitions for blocking public IP addresses and requiring MFA for administrators. These policies were tailored to meet specific security needs, ensuring that resources remain private and secure. The tagging policy was another crucial addition, helping to keep resources organized and accountable, while the location restriction policy ensured deployments align with compliance and performance requirements.



In this stage, I focused on implementing foundational and advanced security measures to enhance the protection and organization of my lab environment. Multi-Factor Authentication (MFA) was a critical step, ensuring all employee accounts had an extra layer of security. For executive-level accounts, like Rachel Platinum’s, I enforced MFA to safeguard high-value resources.

Additionally, I configured essential Conditional Access policies. These included automating MFA enforcement for all users and adding a "High Risk" policy to address specific vulnerabilities like compromised accounts or suspicious login behavior. These policies were tailored to meet security needs while maintaining usability, ensuring resources remained private and secure.

Beyond Conditional Access, I implemented other key configurations:

![[Pasted image 20241226170008.png | 900]]

These measures collectively enhance security by addressing common vulnerabilities and enforcing a layered, real-world approach to securing an Azure AD environment.


## Setting Up Privileged Identity Management (PIM) and Just-In-Time (JIT) Access

In this part of the lab, I focused on implementing Privileged Identity Management (PIM) to enhance security and accountability for elevated roles in Azure AD. PIM enables the use of Just-In-Time (JIT) access, ensuring sensitive roles are only activated when required. This configuration limits activation to specific tasks, such as an 4-hour window requiring multi-level approval and justification. These permissions are granted specifically for tasks that require short-term activation, reducing risks and ensuring smooth operations.

The primary goal of setting up PIM with JIT is to minimize the exposure of privileged roles. High-tier roles like Privileged Role Administrator come with significant permissions, so granting them permanently increases the risk of misuse or accidental errors. This setup aligns with best practices for enforcing the principle of least privilege and strengthens the overall security posture of the environment.



![[Screenshot 2024-12-23 164522.png |600]]


### What I Configured

I configured PIM for **Rachel Platinum**, the Chief of Technology, to ensure her elevated role as Privileged Role Administrator is only active when necessary. The role is assigned with specific settings: approvals from multiple Chief-level groups (HR, Sales, Logistics), an 8-hour activation window, and mandatory MFA verification for activation. To ensure oversight, I set the following configurations:



![[Screenshot 2024-12-23 164740.png|400]]


**Approval Workflow**: I assigned the approval process to all Chief-level groups (HR, Sales, and Logistics). This prevents a single group from becoming a bottleneck and ensures broader accountability. Rachel’s role request must be approved by one of these groups before activation.

**Activation Duration**: I limited role activation to a maximum of 8 hours. This ensures privileges are only granted for short, controlled periods.

![[Screenshot 2024-12-23 171105.png | 600]]


**Annual Review**: I enabled eligibility to expire annually. This allows for periodic auditing to ensure only necessary users retain role eligibility.


![[Screenshot 2024-12-23 171533.png]]



### Testing PIM request

When Rachel logs in and requests the Privileged Role Administrator role, she is directed to this activation page before the permissions are granted. This request must be approved by users in the Chief groups  ensuring that all activations are reviewed and authorized by the appropriate decision-makers.

This process serves a dual purpose. Firstly, it enhances **security** by implementing Just-In-Time (JIT) access, which minimizes the time high-tier privileges are active and reduces the potential for misuse. Secondly, it ensures **accountability** by requiring approvals, providing a clear record of who activated the role and why. This setup aligns with the **principle of least privilege**, limiting unnecessary exposure to sensitive permissions and ensuring they are only granted when genuinely needed.

![[Screenshot 2024-12-23 181630.png]]


The approval process requires members of the Chief-level groups to assess and approve the activation, ensuring thorough verification before elevated permissions are granted. I then logged in as an approver to review and approve the request. The process worked seamlessly, demonstrating the effectiveness of the workflow.

![[Screenshot 2024-12-23 204644.png]]


## PIM for back up Role


As a contingency plan, I decided to configure a backup PIM setup in case the Chief of Technology role becomes vacant. This ensures that if the current user holding the Chief of Technology position is no longer with the company, there is a seamless process for delegating the Privileged Role Administrator access.

To achieve this, I created a dedicated group with no members initially. This group was assigned the Privileged Role Administrator role, and I configured PIM to manage the group's access. I made the Chief of HR eligible as a member of the group through PIM. This allows the Chief of HR to oversee the selection of a new Chief of Technology and assign privileges as needed. Additionally, I included the Manager of Technology as an eligible member, considering that they would be the most logical successor due to their technical expertise and familiarity with the system.

This setup ensures that the Manager of Technology, who likely has the most knowledge of Azure AD and its configurations, can step in if necessary. Meanwhile, the Chief of HR can make an interim appointment or manage role assignments until a permanent replacement for the Chief of Technology is decided.

This way the organization has a structured way to regain access to critical roles and responsibilities in the event of unexpected changes, reducing downtime and minimizing risks associated with the loss of key personnel. This reflects the importance of proactive planning in maintaining security and accountability within the IAM framework.

![[Screenshot 2024-12-23 221151.png]]






## Monitoring Log Setup

![[Pasted image 20241228143804.png]]

In modern Identity and Access Management (IAM) systems, monitoring and alerting capabilities are essential for maintaining security and ensuring operational efficiency. I set up real-time logs and alerts to help detect potential security threats early while providing detailed insights for audits, compliance, and proactive risk management. Monitoring isn't just about catching threats—it's about creating visibility into the system so teams can make informed decisions quickly.

For this stage, I focused on establishing a system in Azure AD to monitor critical events like sign-in failures, risky user behaviors, and audit activities. These tools are vital for ensuring that when something suspicious happens, the right people are notified promptly, enabling them to determine if the event is risky or a normal anomaly. Because my mock company is relatively small, I decided to notify just one person—the Manager of Technology. I believe this position is more in tune with daily activities and can respond effectively to alerts. However, in a larger organization, companies might leverage tools like Microsoft Sentinel, SIEM, or other third-party platforms to build a more robust and scalable monitoring system.

![[Pasted image 20241228151939.png| 800]]


Azure Monitor is a comprehensive, cloud-native monitoring and management solution within Microsoft Azure. It collects, analyzes, and acts on telemetry data from various Azure resources and external environments. Here’s how the components in the diagram work together to deliver a powerful monitoring ecosystem:

#### **1. Data Sources**

Azure Monitor aggregates data from a wide range of sources to provide a unified view of system performance and security:

- **Applications**: Tracks performance and availability of apps, such as response times and error rates, using Application Insights.
- **Operating Systems**: Gathers telemetry data from virtual machines or on-premises servers to detect potential hardware or software issues.
- **Azure Resources**: Monitors services like virtual networks, storage accounts, and databases to ensure optimal performance.
- **Azure Subscriptions and Tenants**: Captures activities across multiple subscriptions or tenants to provide an overarching view of your Azure environment.
- **Custom Sources**: Integrates data from on-premises systems or third-party services using APIs or connected agents.

#### **2. Metrics and Logs**

At the core of Azure Monitor are **metrics** and **logs**, which together provide a complete view of the health and activity of your environment:

- **Metrics**: Real-time numerical data, such as CPU utilization, memory consumption, and request latency. Metrics are ideal for identifying trends and monitoring performance.
- **Logs**: Detailed event records, including user actions, configuration changes, and security events. Logs provide in-depth context for troubleshooting and analysis.

#### **3. Insights and Visualization**

Once the data is collected, Azure Monitor provides tools to make sense of it:

- **Insights**: Specialized monitoring for applications, containers, and virtual machines. For example, Application Insights identifies bottlenecks in web applications.
- **Visualization**: Tools like dashboards, Power BI, and workbooks make data easily digestible. Custom dashboards can be tailored to specific team needs, ensuring everyone has access to relevant information at a glance.

#### **4. Analysis and Responses**

Azure Monitor empowers organizations to analyze data and take proactive measures:

- **Metric Analytics**: Helps identify patterns and anomalies across collected metrics.
- **Log Analytics**: Leverages KQL (Kusto Query Language) for in-depth log querying, enabling root cause analysis and audit tracking.
- **Alerts**: Automatically notifies users about unusual behavior, such as exceeding resource limits or detecting suspicious activities.
- **Autoscale**: Dynamically adjusts resources to meet changing workloads, helping manage costs and maintain service reliability.

#### **5. Integration and Extensibility**

Azure Monitor integrates seamlessly with other Azure services and external platforms for enhanced functionality:

- **Logic Apps**: Automates workflows based on monitoring insights. For instance, you can trigger incident responses or send notifications automatically.
- **Export APIs**: Shares telemetry data with third-party systems like SIEM tools, enabling robust, end-to-end monitoring for hybrid environments.

![[Pasted image 20241228153238.png|800]]


I started by creating a workbook to store and analyze the data logs. In Azure Monitor, a workbook is an easy-to-use tool that helps visualize and analyze log data in a customizable way. It’s perfect for building dashboards, reports, and queries that offer clear insights into this system without the need for complicated setup or additional infrastructure.

For larger organizations or more complex environments, tools like Log Analytics Workspaces, Microsoft Sentinel, or third-party SIEM solutions like Splunk are often preferred. These options provide advanced analytics, scalability, and integration capabilities for handling larger datasets and diverse use cases. However, because my mock company is small, I decided to use a workbook. It’s cost-effective, simple to set up, and fulfills my immediate needs for setting up alerts and visualizing key data points efficiently.

![[Screenshot 2024-12-27 150625.png | 600]]


### Diagnostic Settings: Capturing Critical Data for Analysis

Diagnostic Settings in Azure Monitor are designed to collect and forward logs and metrics from Azure resources. They allow you to choose which types of data to track—such as sign-ins, audit logs, and risky users—and where to send it for storage and analysis. In my setup, I configured Diagnostic Settings to send key logs to the Log Analytics Workspace, where they are stored for querying and analysis

#### **How It work with Workspace

The Diagnostic Settings route critical log categories—like sign-ins and risky user data—to the Log Analytics Workspace. This combination provides near-real-time insights into critical IAM activities while maintaining a simple and cost-effective setup for my small company

![[Screenshot 2024-12-28 174335.png| 600]]


Here’s why I selected specific logs:

- **Audit Logs**: Track changes to Azure AD resources, such as user roles and group modifications.
- **Sign-In Logs**: Record all login attempts, helping detect unauthorized access or suspicious login patterns.
- **Risky Users**: Identify potentially compromised accounts flagged by Azure’s machine-learning models.
- **User Risk Events**: Provide details about the events contributing to a user’s risk score, such as leaked credentials or sign-ins from risky IP addresses.
- **Risky Service Principals**: Monitor risky behavior associated with service principals, adding a layer of security for future application and service interactions.


### Alert rules

Alert rules in Azure Monitor are like a watchful eyes over my environment, helping me stay proactive about potential issues. These rules let me define specific conditions—like unusual login attempts or resource usage—and take action when those conditions are met.

For example, I can set up alert rules to notify me when someone repeatedly fails to sign in or when a resource is overused. Essentially, these rules help me keep track of system health, performance, and security threats.

- **What They Do:** They monitor logs or metrics and trigger alerts when a defined threshold is crossed.

I navigated to the **IAM-Monitoring-Workspace** in my **Log Analytics Workspace**, which I had previously set up. Once there, I opened the **Alerts** tab to create an alert rule that would notify me if a user failed to sign in five times within five minutes.

![[Screenshot 2024-12-28 222628.png | 700]]

This query helps me detect potential brute-force attacks or suspicious login patterns. 

```
SigninLogs
| where TimeGenerated > ago(5m)
| summarize FailedSignInCount = count() by UserPrincipalName, IPAddress
| where FailedSignInCount > 5
```

The output includes the user's name (UserPrincipalName) and the originating IP address, providing essential details to investigate the event further.

![[Screenshot 2024-12-28 195546.png | 800]]

To ensure alerts are monitored effectively, I assigned the Manager of Technology to receive them. This choice was made because the Manager is deeply involved in daily IT operations, making them well-suited to address potential issues promptly and efficiently.

To further secure this alert, I decided to lock the rule to prevent accidental deletion. Locking the alert rule ensures that critical monitoring remains intact and cannot be mistakenly removed by anyone with access to the system.

![[Screenshot 2024-12-28 202909.png]]


### Testing the Alert


To validate the alert rule, I tested it by deliberately causing failed login attempts. I tried signing in with incorrect credentials multiple times, triggering the conditions outlined in the rule. Afterward, I checked the IAM-Monitoring-Workspace under the Alerts tab and confirmed that the alert had fired as expected. The system accurately logged the failed attempts and raised the alert, demonstrating that the rule is functioning as intended and providing critical visibility into potential security threats.


![[Screenshot 2024-12-28 204601.png]]

![[Screenshot 2024-12-28 215341.png]]


Beyond monitoring failed sign-ins, I set up additional alert rules that match with selected alerts. 

![[Pasted image 20241230190821.png]]


The monitoring setup demonstrated the effectiveness of Azure Monitor in securing and managing environments. I created a centralized workspace that aggregated critical logs including AuditLogs, SignInLogs, RiskyUser logs, and UserRiskEvents. This ensured that all IAM-related activities were tracked efficiently.

Additionally, I implemented alert rules to detect anomalies such as failed sign-in attempts exceeding thresholds. These measures enhanced system responsiveness and provided actionable insights for mitigating risks. By focusing on these key configurations, this project highlights the importance of visibility and proactive monitoring in IAM systems, laying the groundwork for scalable and efficient security practices.


## ***Conclusion***

his stage of the lab was about taking the foundation I built and turning it into something truly secure and practical. I focused on smart, layered security—things like MFA, Conditional Access, and Privileged Identity Management—ensuring users get just the access they need, no more, no less. These additions are great feature for a foundation that will help me build forward into more complex setup.



![[Security framwork.png |700]]


Adding custom Azure Policies helped keep resources secure and compliant, while monitoring and alerts gave me visibility to catch potential issues early. The goal wasn’t just locking things down—it was making security feel seamless and natural, so it protects without slowing anyone down.

At its core, this stage is about building something flexible. I’ve set the stage for adding apps, handling new workflows, or scaling up as needed. This lab isn’t just a showcase of what I’ve done; it’s a starting point for what I can do next.
