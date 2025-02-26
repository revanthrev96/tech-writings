
### Simulating Cloud Attacks with Stratus Red Team

Tired of manually creating Proof of Concepts (POCs) for detections? There’s a smarter way! I recently came across an awesome approach that can make your life easier.

Many of you might already be familiar with using simulation tools to build effective detection POCs. One of the most efficient ways to detect cloud-based attacks is with **Stratus Red Team** — an open-source tool designed for simulating realistic attacks on **Cloud** and **Container** environments. 🌐

For more details, check out the official documentation. But first, let’s go through some prerequisites to get started:

✅ **Valid AWS IAM User Credentials** — (For security reasons, avoid using the root account!) ✅ **Latest Version of Go** ✅ **Stratus Red Team tool** — (You’ll need this!) ✅ **Kali Linux** — (I personally prefer Kali, but feel free to use any OS you’re comfortable with.)

### Setting Up IAM User 🔐

First, I’m creating an **IAM user** on my AWS account. One key thing to note — if your account is critical, avoid using it! Instead, try a **sandbox/demo account**. If it’s a **free-tier** account, that’s an added advantage.

To proceed:

1.  **Create an IAM User** with **Administrative Access** permissions.
2.  Generate **Access Key** and **Secret Key** — store them safely for future use.

In my case, I’m using the username **“DETest”** with **Admin Permission**.

![](https://cdn-images-1.medium.com/max/720/1*iHGyPlfvguRLOfKurLnH2g.png)

IAM User

### Configuring AWS CLI ⚙️

Once the IAM user is created, the next step is to configure the credentials in AWS CLI. There are two common methods to do this:

1.  **Using the terminal:** Run `aws configure` and enter the required details, including Access Key, Secret Key, region, and output format.
2.  **Manually updating configuration files:**

-   Add credentials to `~/.aws/credentials`:
-   `[default]  
    aws_access_key_id=YOUR_ACCESS_KEY aws_secret_access_key=YOUR_SECRET_KEY`
-   Set the region and output format in `~/.aws/config`:
-   `[default]  
    region=YOUR_REGION  
    output=json`

![](https://cdn-images-1.medium.com/max/720/1*0U296w4UTNNWHrsw1UT7dQ.png)

### Installing Go 🛠️

Now that the credential setup is complete, let’s move to the next step — installing **Go**.

To check if Go is already installed, run:

go version

Ensure that you’re using **version 1.22+**, as older versions may cause dependency errors during installation.

If Go is not installed, download it from the official site: 🔗 [Go Official Download](https://go.dev/)

⚠️ Always make sure you’re downloading from the official website to maintain security best practices.

Once downloaded, proceed with the installation and set it up in your environment.

![](https://cdn-images-1.medium.com/max/720/1*B0rV_l_TKt-rpswQrZ_k_w.png)

### Installing Stratus Red Team ⚡

All good so far! Now, let’s proceed with our next step — installing **Stratus Red Team**.

To install it, please use the [**GitHub repository**](https://github.com/DataDog/stratus-red-team) link.

After installation, verify if the tool is working correctly by running:

stratus list

If everything is set up properly, you should see a list of available attack techniques. However, if an error appears, troubleshoot and fix it accordingly.

![](https://cdn-images-1.medium.com/max/720/1*FYIVLryC208QAhAxP7EHzg.png) 

### Simulating an Attack 🎯

Now that we have all prerequisites set, it’s time to simulate an attack.

For this demo, we’ll use the technique **aws.exfiltration.ec2-share-ebs-snapshot**, which simulates exfiltrating an **EBS snapshot** by sharing it with an external AWS account.

![](https://cdn-images-1.medium.com/max/720/1*LU_79dbyhOE50F6sMgp3TQ.png)

#### Understanding Stratus Attack States

Stratus operates in three states:

-   **Cold** — Attack has not been executed.
-   **Warm** — Environment is being prepared for attack execution.
-   **Hot** — The attack is detonated.

Currently, we’re in the **Cold State**, meaning no attack has been initiated yet.

![](https://cdn-images-1.medium.com/max/720/1*-P654IJF-R4gioN-W1ATIw.png)

#### Setting Up the Attack Environment 🛠️

To move forward, we transition to the **Warm State** by running:

stratus warmup aws.exfiltration.ec2-share-ebs-snapshot

At this stage, Stratus verifies authentication and uses **Terraform** in the background to set up attack prerequisites.

![](https://cdn-images-1.medium.com/max/720/1*4i63YNbAKlqbc1QZsQLdPw.png)

Once complete, you should see logs indicating the creation of **Volume** and **Snapshot** resources. You can verify this using **AWS CLI** or directly by checking **CloudTrail logs**.

![](https://cdn-images-1.medium.com/max/720/1*QQ4O6me3Jg-Sv8IQ1bJLpw.png)

#### Executing the Attack 🚨

To fully execute the attack, we transition to the **Hot State** by running:

stratus detonate aws.exfiltration.ec2-share-ebs-snapshot

![](https://cdn-images-1.medium.com/max/720/1*tWc0QYLhAE0qxz8IRn7eXg.png)

After detonating, check **CloudTrail logs** for the `ModifySnapshotAttribute` API event. This confirms that the snapshot has been shared with an external AWS account.

![](https://cdn-images-1.medium.com/max/720/1*ToUKkF_S4VSkbVSEQboxXA.png)

![](https://cdn-images-1.medium.com/max/720/1*HrOkdkWk83Jf-uoAbLtWEQ.png)

#### Rolling Back & Cleanup 🧹

After execution, it’s important to **roll back** the attack changes. First, revert to the **Warm State**:

stratus revert aws.exfiltration.ec2-share-ebs-snapshot

![](https://cdn-images-1.medium.com/max/720/1*jtDTcgOhr7NLUykTiFp88w.png)

Then, verify that the snapshot is no longer shared externally.

![](https://cdn-images-1.medium.com/max/720/1*3eYGM6fk_MqAfcIrUUwkcg.png)

Finally, **clean up resources** to avoid unnecessary AWS costs:

stratus cleanup aws.exfiltration.ec2-share-ebs-snapshot

![](https://cdn-images-1.medium.com/max/720/1*0BXdtf8IjoBxuHYmNJy3hA.png)

![](https://cdn-images-1.medium.com/max/720/1*oPoJn2bXnLR1hDvkTRvaqA.png)

We’ve successfully simulated an **EBS snapshot exfiltration attack**, verified it via **CloudTrail**, and cleaned up our environment. This approach helps security teams test and refine their detection capabilities in a controlled setting.

### Detection Analysis 🛡️

Now that we’ve successfully simulated the attack, it’s time to focus on **detection strategies**. To detect this attack, you can use any **SIEM tool** and upload the simulated logs. If you’re comfortable with **AWS Athena**, you can analyze the logs there. The choice of tools depends on your organization’s detection environment.

#### Step 1: Understanding the Attack & Impact

Before developing any detection rules, it’s essential to understand the **attack flow** and its potential impact.

📌 **What happens in this attack?**  
The **EBS Snapshot Sharing Attack** is a technique where an attacker creates an **EBS snapshot** and then shares it with an external AWS account, leading to potential **data exfiltration**.

📌 **Attack Steps:**

1.  The attacker **creates a snapshot** from an existing EBS volume (which may contain sensitive data).
2.  They **modify snapshot permissions** to allow another AWS account access.
3.  This is done using the **ModifySnapshotAttribute API**, which grants `createVolumePermission`.
4.  The attacker’s AWS account can now **see and create a new volume** from this snapshot.
5.  They attach the volume to an EC2 instance and **extract the data**.

#### Step 2: Identifying Key Detection Indicators

To detect this attack, look for the following events in your SIEM or CloudTrail logs:

✅ **Suspicious API Calls**

-   Monitor **ModifySnapshotAttribute** API calls, as this is used to share snapshots externally.
-   Look for `requestParameters.createVolumePermission.add.items{}.userId`—this contains the AWS account ID(s) that the snapshot is shared with.

✅ **Unexpected Snapshot Activity**

-   Track **SharedSnapshotCopyInitiated** (when an attacker copies the snapshot to their account).
-   Track **SharedSnapshotVolumeCreated** (when an attacker creates an EBS volume from a shared snapshot).

✅ **OSINT & User Behavior Analysis**

-   Check **IP reputation** and perform **OSINT checks** to validate if the request originated from a suspicious entity.
-   Analyze **User-Agent** strings — while they can be manipulated, it’s still worth investigating.
-   Identify the user performing these actions — do they have **excessive permissions**?

#### Step 3: Preventing & Mitigating Future Attacks

Once you’ve developed detections, it’s equally important to **prevent** such incidents in the future. Here are some recommendations:

🚨 **Limit who can modify snapshot attributes** — restrict access to the **ModifySnapshotAttribute** API.  
🔒 **Use encrypted snapshots** — AWS prevents unauthorized accounts from accessing encrypted snapshots.  
⚠️ **Implement IAM least privilege** — ensure users and roles only have the permissions they absolutely need.

#### Step 4: Handling False Positives

When implementing **custom detections**, false positives are inevitable. To improve detection accuracy:

-   Maintain an **exclusion table** to filter out known benign activities.
-   Use **anomaly detection** instead of static rules for behavioral-based alerts.
-   Continuously refine your detection logic based on real-world attack patterns.

By following this approach, you can **effectively detect, investigate, and mitigate** snapshot-sharing attacks in AWS. Stay vigilant and keep improving your detection capabilities! 🚀

### Connect with Me 🔗

If you found this blog helpful and want to discuss more on **cloud security and detections**, feel free to connect with me:

🔹 [**LinkedIn**](https://www.linkedin.com/in/revanth-s-15a410b7/)  
🔹 [**GitHub**](https://github.com/revanthrev96)

Let’s collaborate and learn together! 🚀
