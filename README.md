# Security Monitoring System on AWS

#### 🔐📊 Project Storyline 🔍🛡️

At **FinPay**, a fintech company, sensitive credentials are securely stored in **AWS Secrets Manager**.

One morning, a developer unintentionally accesses a secret during a routine script run. **AWS CloudTrail** immediately logs the activity and sends it to both:
- An **S3 bucket** for archiving
- A **CloudWatch Logs** group for real-time monitoring

Within CloudWatch, a **metric filter** detects the secret access event and updates a **custom metric**. A **CloudWatch Alarm** linked to this metric goes into an alarm state. This triggers an **Amazon SNS topic** subscribed by the security team.

Within moments, the security team receives an **email alert** detailing:
- Who accessed the secret
- When it happened
- From where

They quickly verify the action and determine it was unintentional. The system worked exactly as designed — **detecting, logging, alerting**, and enabling a rapid response.

FinPay continues operating securely, knowing **secret access is under watch 24/7**.
![security](https://github.com/user-attachments/assets/1aabad2d-ae38-4a82-b25b-f78701c2f59a)

## AWS Services Involved

| **Service**               | **Purpose**                                             |
|---------------------------|---------------------------------------------------------|
| **AWS Secrets Manager**   | Stores sensitive credentials                            |
| **AWS CloudTrail**        | Captures API calls (activities done by users)           |
| **Amazon S3**             | Stores CloudTrail logs for auditing                     |
| **CloudWatch Logs**       | Central log aggregation and filtering                   |
| **CloudWatch Metric Filters** | Detects specific patterns like secret access        |
| **CloudWatch Alarms**     | Triggers on metric thresholds                           |
| **SNS**                   | Sends alert notifications                               |
| **Email (via SNS)**       | Delivers alert to SecOps team                           |

---


## Project Details!
[Uploading security.jpg…]()

### Stage 1

**Setup AWS Secrets Manager and create a secret**

- **Key**:  
  The key is a label or identifier used to describe a specific piece of data within a secret, like `"password"` or `"apiKey"`.  
  For this demo: `secret_for_demo_security_project`

- **Value**:  
  The value is the actual sensitive information stored under the key.  
  For this demo: `mysecretDem0Pr0Jec,ttt`

![Screenshot 2025-04-14 122027](https://github.com/user-attachments/assets/90096693-89bc-4b15-aa76-1cf34c1cce98)
- **Name of the secret**:  
  The name of the secret is a unique identifier (like `"prod/db-credentials"`) used to reference the entire secret in **AWS Secrets Manager**.  
  For this demo: `mydemosecret`
![Screenshot 2025-04-14 124319](https://github.com/user-attachments/assets/de1a8b38-d6b7-4436-8b4f-110e19d5a949)
![Screenshot 2025-04-14 130158](https://github.com/user-attachments/assets/cc132c31-0a09-496b-a07c-31cb85f7b5de)

**Setup AWS CloudTrail to log all API calls made in the AWS account, including access to secrets.**

- **CloudTrail** is enabled by default to track all management-level API calls in the AWS account with a log retention period of **30 days**.

- However, creating a **Trail** allows you to:
  - Track both **management and data-level** API calls
  - Set a **longer retention period** by storing logs in an **S3 bucket**

#### 📌 Steps:
1. Go to **CloudTrail** in the AWS Console
2. Click **Create a Trail**
3. Follow the prompts and select **Create Trail**
![Screenshot 2025-04-14 131114](https://github.com/user-attachments/assets/b62b730d-e3ad-45f2-b110-1996b5c1ccc0)
![Screenshot 2025-04-14 131640](https://github.com/user-attachments/assets/537235e5-739d-4cec-b3f4-2c8d7f44008d)

#### ✅ Confirmation

After creating the trail, the **first confirmation** is the **CloudTrail status** indicating that **logging is enabled**.
This confirms that CloudTrail is actively recording API activity across your AWS account.

### Stage 2 (continued)

**Setup CloudWatch filters to find logs in CloudTrail relating to access to Secrets Manager.**
To achieve this:
1. Click on the **CloudTrail event** (the trail you created earlier).
2. Scroll down to the **CloudWatch Logs** section.
3. Click on **Edit** to modify or configure the log group destination and ensure CloudTrail is streaming logs to CloudWatch.

This step enables real-time visibility and filtering of API calls — including those made to **AWS Secrets Manager**.
![Screenshot 2025-04-14 133406](https://github.com/user-attachments/assets/acee10ee-1220-4f6f-97c3-3c2e2c86bf9b)
#### 🔧 Configure CloudWatch Logs for the Trail

1. **Check the "Enabled" box** to allow CloudTrail to send logs to CloudWatch Logs.

2. **Log Group**:
   - Leave the log group as **"New"** unless there are existing log groups specifically created for CloudTrail events.
   - A **log group** in **AWS CloudWatch** is a container that organizes and stores logs from the same application, service, or resource.
   - Rename the default log group to something simpler and descriptive.  
     _Example_: `LogGroupForCloudTrailEvent`

3. **IAM Role**:
   - CloudTrail needs permission to deliver logs to CloudWatch Logs.
   - This is done using an **IAM role** that CloudTrail will assume.
   - Click **New** and change the role name to something like:  
     `CloudTrail_CloudWatchLogs_Role`
   - If the role already exists, select **Existing** and choose the appropriate role from the dropdown.

Once this is set, CloudTrail will begin streaming logs directly into your chosen CloudWatch Logs group, making it possible to filter for sensitive actions like secret access.
![Screenshot 2025-04-14 135136](https://github.com/user-attachments/assets/610470e7-7f7c-4c26-b136-fba076b44aa9)

#### ✅ Verify Changes

1. **Confirm Log Group Creation**:
   - Navigate to the **CloudWatch** service in the AWS Console.
   - Go to **Logs** → **Log Groups**.
   - Ensure that your newly created log group appears in the list of CloudWatch Logs groups.
This confirms that your CloudTrail events are successfully being logged to CloudWatch, and you can now filter these logs for specific events like secret access.
![Screenshot 2025-04-14 135754](https://github.com/user-attachments/assets/a79acb0b-d0d8-4f8a-a813-59f2aefdf06d)

2. **View Log Group Details**:
   - Click on the log group you just created to view more details.
   - In the log group, you'll find **log streams**. 
   
   **Log Streams**:
   - Log streams are created per CloudTrail event delivery batch. These are typically organized based on **time intervals** or the **volume of events**.
This allows you to drill down into specific batches of events and analyze the logs in more detail.
![Screenshot 2025-04-14 135856](https://github.com/user-attachments/assets/5ed29d76-8eb4-4b31-9026-9c42f62df83c)

3. **View Event Details in Log Streams**:
   - Click on any of the **log streams** to get a detailed list of events that CloudTrail has sent to CloudWatch Logs.
   
   Each log stream contains a list of **CloudTrail events** with detailed information, such as:
   - The event name (e.g., `GetSecretValue` for Secrets Manager access)
   - The source IP address
   - The AWS account ID
   - The time of the event
This step lets you dive deeper into the specifics of what actions were taken within your AWS environment, especially concerning sensitive resources like Secrets Manager.
![Screenshot 2025-04-14 140210](https://github.com/user-attachments/assets/f8572b17-8b4c-4d1a-b2f0-258b24656f68)

### Stage 3: Configure SNS for Alarm Notifications

1. **CloudTrail Event Details**:
   - The **dropdown** for each event in the log stream will provide as much detail as possible for every event logged by CloudTrail. This includes detailed information about the API calls, the resources accessed, and other relevant metadata.

2. **Configure SNS Topic to Receive Alarms**:
   - Go to **Amazon SNS** in the AWS Console.
   - Click on **Topic** → **Create Topic**.
   - Select **Standard** for the topic type.
   - Provide a **Topic Name**.  
     _Example_: `NotificationForCloudTrail`
   - Click **Create Topic**.
This SNS topic will be used to send notifications when CloudWatch Alarms are triggered, keeping your team informed in real-time of any suspicious activity related to sensitive resources like AWS Secrets Manager.
![Screenshot 2025-04-14 141241](https://github.com/user-attachments/assets/779db29a-92e4-4e3f-a8ed-b60caba8335f)

### Stage 4: Create a Metric Filter for Secrets Manager Access

1. **Navigate to CloudWatch Logs**:
   - Go to **CloudWatch** in the AWS Console.
   - Under **Log Groups**, click on the log group you created earlier, e.g., `LogGroupForCloudTrailEvent`.
   - Click on **Actions** and then select **Create a Metric Filter**.

2. **Define Filter Pattern**:
   - Under the **Filter Pattern** field, enter the pattern to search for specific API calls.  
     The key pattern for detecting Secrets Manager access is `"GetSecretValue"`, which is the API call used to retrieve the encrypted value of a secret from **AWS Secrets Manager**.
   
   - **Explanation**:  
     - **"GetSecretValue"** is the API call that CloudTrail logs whenever a secret is accessed, whether from the Console, SDK, script, or CLI.
     - This metric filter will ask CloudWatch to scan all CloudTrail logs for any occurrence of **"GetSecretValue"** and trigger an alarm when it’s found.
This setup helps detect when sensitive secrets are accessed and allows you to create an alert based on those actions.

![Screenshot 2025-04-14 143710](https://github.com/user-attachments/assets/ed262841-9cde-48e3-9ad6-0e7118249bc0)
3. **Name the Metric Filter**:
   - Give the filter a meaningful name, for example:  
     `FilterForSecretsAccess`

4. **Define Metric Parameters**:
   - **Metric Namespace**:  
     This is a container or category for related metrics.  
     _Example_: `SecurityMetrics`

   - **Metric Name**:  
     The specific name of the metric you're tracking.  
     _Example_: `MetricsForSecretsAccess`

   - **Metric Value**:  
     This is the value that CloudWatch will assign whenever a log entry matches the filter (e.g., when the `GetSecretValue` API call is made).  
     Set this value to **1**.  
     - Every time the `GetSecretValue` API is called, CloudWatch will increment the metric by 1.

   - **Default Value**:  
     Set this to **0**.  
     - If no log events match your filter in a given period, CloudWatch will still publish a metric with a value of **0**.

Once this is done, you’ll have a custom metric that tracks every instance of `GetSecretValue` being called and triggers an alarm when the threshold is crossed.
![Screenshot 2025-04-14 145056](https://github.com/user-attachments/assets/a1331d79-99f7-47cc-b4c5-c41696d751ac)

### Stage 5: Create Metric and Configure CloudWatch Alarm

1. **Create the Metric**:
   - After defining the metric filter (as described earlier), CloudWatch will automatically create a custom metric.
   
2. **Configure CloudWatch Alarm**:
   - Access the **log group** you created for CloudTrail events (e.g., `LogGroupForCloudTrailEvent`).
   - Click on the **metric filter** that you created earlier to view the custom metric.
   - Check the box to select the metric.

3. **Set Up Alarm**:
   - Once the metric is selected, click on the **Create Alarm** button.
   - Configure the alarm to trigger when the number of `GetSecretValue` API calls crosses a defined threshold.
   - In the **Conditions** section, define the threshold for your alarm (e.g., when the metric value is greater than 0 or exceeds a certain number of calls).
   
4. **Use SNS Topic for Notifications**:
   - In the **Actions** section, choose to **Send a notification to an SNS topic**.
   - Select the **SNS topic** (e.g., `NotificationForCloudTrail`) that you created earlier.
   - This ensures that an email alert will be sent whenever the number of secret access events (matching the `GetSecretValue` API call) crosses the defined threshold.

This will trigger an **email alert** via SNS whenever the `GetSecretValue` API call exceeds the set threshold, providing real-time monitoring of sensitive secret access in your AWS
![Screenshot 2025-04-14 150315](https://github.com/user-attachments/assets/5a93a6a6-0437-4384-b799-7b4834e83325)

5. **Create the Alarm**:

   - On the left-hand side of the alarm configuration page, change the **Statistic** to **Average** and set the **Period** to **5 minutes**.

   **Explanation**:
   - **Statistic**: Setting the statistic to **Average** will cause the alarm to evaluate the average metric value over each 5-minute period.
   - **Period**: The **5-minute period** ensures that short-term spikes are smoothed out, and the alarm will track sustained activity patterns (e.g., frequent `GetSecretValue` API calls over time) instead of just reacting to a single burst of activity.

This configuration helps you detect **ongoing patterns** of secret access, reducing false alarms triggered by short, isolated spikes in activity.
![Screenshot 2025-04-14 150530](https://github.com/user-attachments/assets/1a25445b-8322-412d-ae7d-80633dd54a09)

6. **Configure the Alarm Condition**:

   - Set the **Threshold Type** to **Static**.
   - Configure the condition as:  
     **Whenever `MetricsForSecretsAccess` is Greater/Equal to 1**.

   **Explanation**:
   - The alarm will be triggered whenever the **`MetricsForSecretsAccess`** metric exceeds or equals **1**.
   - This means that the alarm will fire as soon as **one instance** of the `GetSecretValue` API call is logged, ensuring that the system reacts promptly to any access to sensitive secrets.

This setup ensures the alarm is triggered when the `GetSecretValue` API is called at least once, providing immediate alerts for secret access events.
![Screenshot 2025-04-14 151257](https://github.com/user-attachments/assets/75429a9b-896d-431e-a288-9237624bcd7e)

7. **Configure Notification Actions**:

   - **Set the State to In-Alarm**:  
     This setting ensures that when the CloudWatch alarm is triggered (because the metric has crossed your specified threshold, such as a high number of `GetSecretValue` API calls), the alarm will go into the **in-alarm** state.  
     When the alarm is "in-alarm," it indicates that the threshold has been breached, and immediate action is needed.

   - **Create an SNS Topic**:
     - Create a new **SNS topic** to handle the notifications.
     - Name the SNS topic something descriptive.  
       _Example_: `SNSTopicForCloudWatchAlarm`
   
   - **Add Email Address**:  
     - Add the email address of anyone on the security or operations team who needs to be alerted when CloudWatch detects an anomaly (e.g., frequent access to secrets).
   
   - Click **Create Topic** to finalize the SNS topic creation.

This ensures that when the CloudWatch alarm goes into the "in-alarm" state, an email notification will be sent to the team immediately.
![Screenshot 2025-04-14 152948](https://github.com/user-attachments/assets/1ec5f43a-d949-445d-93ec-df3ddbcbde2c)

8. **Name and Finalize the Alarm**

- **Give the Alarm a Name**:  
  _Example_: `AlarmForSNSTopic`

- **Review and Confirm**:
  - Review all the configurations (metric, threshold, notification).
  - Click **Create Alarm** to complete the setup.

---

### Stage 6: Confirm SNS Subscription

After setting up the alarm and notification:

1. **Confirm Email Subscription**:
   - Go to the **SNS** service in the AWS Console.
   - Navigate to **Subscriptions**.
   - Click on the SNS topic you created earlier (e.g., `SNSTopicForCloudWatchAlarm`).
   - Click **Request Confirmation** if it hasn’t already been sent.

2. **Check the Email Inbox**:
   - Access the email account you added to the SNS topic.
   - Look for an email from **AWS Notifications** and click **Confirm Subscription**.
   - This step ensures that the email recipient is aware of and authorizes the subscription to receive future alarm notifications.

---

### Stage 7: Test the Setup

1. **Access the Secret in AWS Secrets Manager**:
   - Go to **AWS Secrets Manager** and retrieve the secret (e.g., `mydemosecret`).
   - This triggers a `GetSecretValue` API call.

2. **Wait and Observe**:
   - Wait for about **5 minutes** to allow CloudTrail to log the event and CloudWatch to process the metric.
   - You should see:
     - The event logged in **CloudTrail**
     - A notification email from **CloudWatch Alarm** sent via **SNS**, alerting about the secret access

This confirms that your monitoring and alerting pipeline is fully functional — logging, filtering, alerting, and notifying your security team in real-time when sensitive secrets are accessed.

```
