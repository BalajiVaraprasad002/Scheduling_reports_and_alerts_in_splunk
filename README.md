# Scheduling Reports and Alerts in Splunk: A Practical Guide

A comprehensive guide to configuring, scheduling, and troubleshooting reports and alerts in Splunk to monitor security events like failed login attempts.

[View the Original PDF](Scheduling%20reports%20and%20alerts%20in%20splunk)

---

## Overview

In Splunk, scheduled reports and alerts are essential for proactive monitoring and automated data analysis. This guide provides a practical, step-by-step walkthrough for setting up a system to track failed login attempts. It covers:

-   Creating and scheduling a daily report for failed logins.
-   Configuring a real-time alert to trigger when failed login attempts exceed a specific threshold.
-   Troubleshooting and refining Splunk Processing Language (SPL) queries for accurate results.
-   Verifying that the alerts and reports function as expected.

## Data Sources Monitored

Splunk can monitor a wide variety of machine-generated data. For this guide, the primary data source is Splunk's internal audit index, which tracks user authentication and administrative actions.

-   **Primary Index:** `index=_audit`
-   **Key Events:** Failed login attempts (`action="login attempt" info="failed"`)

Other common data sources that can be monitored include:
-   System Logs (Linux `syslog`, Windows Event Logs)
-   Security Logs (Firewalls, IDS/IPS, Antivirus)
-   Application Logs (Apache, Nginx, custom apps)
-   Network Traffic Data (NetFlow, SNMP)
-   Authentication and Access Logs (Active Directory, LDAP, Okta)
-   Cloud Services Logs (AWS CloudTrail, Azure Monitor, GCP Logging)
-   Web Server Logs (IIS, Apache)

---

## Practical Guide: Scheduling a Failed Logins Report

This section details how to create a report that runs daily to summarize all failed login attempts over the last 24 hours.

### Step 1: Create the Search Query

1.  Navigate to the **Search & Reporting** app in Splunk.
2.  Set the time range to **Last 24 hours**.
3.  Enter the following SPL query to find all failed login attempts and count them by user and time:
    ```spl
    index=_audit action="login attempt" info="failed" | stats count by user, _time
    ```

### Step 2: Save the Search as a Report

1.  After running the query, click **Save As -> Report**.
2.  Provide a **Title** (e.g., `audit_failedlogins_report`) and an optional **Description**.
3.  Save the report.

### Step 3: Schedule the Report

1.  After saving, view the report and click **Edit -> Edit Schedule**.
2.  Check the **Schedule Report** box.
3.  Configure the schedule to **Run every day**.
4.  Set the **Time Range** to **Last 24 hours**.
5.  Set the **Schedule Priority** as needed (e.g., Default or Higher).
6.  Optionally, set a **Schedule Window** to allow for execution delays.

### Step 4: Configure Email Notifications

1.  Under **Trigger Actions**, click **Add Actions -> Send email**.
2.  Enter the recipient email address(es) in the **To** field.
3.  Set the **Priority** to High to ensure visibility.
4.  Customize the **Subject** and **Message**.
5.  Choose to include results by attaching a **PDF** or **CSV**.
6.  Click **Save** to finalize the scheduled report.

---

## Practical Guide: Creating a Failed Logins Alert

This section explains how to create a real-time alert that triggers when a user has more than 3 failed login attempts.

### Step 1: Create the Alert-Specific Query

The initial query for the alert must be refined to only return results when a specific condition is met. The key is to **group attempts by user** and then filter based on the count.

**Initial Flawed Query (for demonstration):**
The guide first shows a common mistake where `_time` is included in the `stats` command. This fragments the results by timestamp, preventing the count from accumulating correctly per user.
```spl
# Incorrect - Will not group attempts correctly
index=_audit action="login attempt" info="failed" | stats count as failed_attempts by user, _time | where failed_attempts > 3
```

**Corrected and Final Query:**
Remove `_time` to correctly group all failed attempts by `user`. This ensures the count is accurate.
```spl
index=_audit action="login attempt" info="failed" | stats count as failed_attempts by user | where failed_attempts > 3
```

### Step 2: Save the Query as an Alert

1.  Run the corrected query in the **Search & Reporting** app.
2.  Click **Save As -> Alert**.
3.  Enter a **Title** (e.g., `_audit_failedlogins_alert`).
4.  Set **Permissions** (e.g., Private).
5.  For **Alert Type**, select **Real-time**.
6.  Set an **Expiration** time (e.g., 24 hours).

### Step 3: Configure Trigger Conditions

1.  For **Trigger alert when**, select **Per-Result**. This ensures the alert fires for each user who crosses the threshold.
2.  Optionally, configure **Throttle** to prevent alert fatigue from the same event.

### Step 4: Configure Alert Actions

1.  Under **Trigger Actions**, click **Add Actions**.
2.  Select **Add to Triggered Alerts** to make the alert visible in the Splunk UI.
3.  Set the **Severity** level (e.g., High).
4.  You can also add other actions like **Send email** or **Run a script**.
5.  Click **Save** to activate the alert.

### Step 5: Verification

1.  Simulate multiple failed login attempts for a single user to test the alert.
2.  Navigate to **Settings -> Searches, Reports, and Alerts**.
3.  You will see the created alert and report. The **Alerts** column for your new alert will show a count greater than zero, confirming it has been triggered successfully.

---

## Alert Actions and Escalation

Once an alert is triggered, Splunk can perform several actions:

-   **Email Notification:** Send detailed alert information to a security team.
-   **Webhook Integration:** Post data to external services like Slack, PagerDuty, or ServiceNow to create incidents or notify on-call personnel.
-   **Custom Scripts:** Execute a script to perform an automated response, such as temporarily disabling a user account.

For robust escalation, integrate Splunk alerts with ticketing or on-call management systems to ensure issues are tracked, assigned, and resolved based on defined SLAs.

---

## Author

**Balaji Varaprasad**

## Connect with Me

-   **LinkedIn:** `https://www.linkedin.com/in/balaji-varaprasad?utm_source=share&utm_campaign=share_via&utm_content=profile&utm_medium=android_app`
