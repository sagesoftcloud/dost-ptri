# Lab 3 — SNS Fan-Out Notifications

## Goal
Build a notification fan-out system where a single event (new user signup) is published to an SNS topic, which then delivers the message to multiple subscribers simultaneously: an email notification, an SQS queue for logging, and a Lambda function for processing.

## Services Used
- **Amazon SNS** — Pub/sub topic
- **Amazon SQS** — Subscriber queue (audit log)
- **AWS Lambda** — Subscriber function (welcome email logic)
- **Email** — Subscriber (direct notification)

## Architecture

```
                          ┌──→ Email (you@email.com)
                          │
[Lambda Publisher] → [SNS Topic] ──→ [SQS Queue] (audit log)
                          │
                          └──→ [Lambda Subscriber] (welcome processor)
```

## Naming Convention
Replace `[yourname]` with your name (e.g., `juan`).

---

## Step 1: Create the SNS Topic

1. Go to **SNS Console** → [https://console.aws.amazon.com/sns](https://console.aws.amazon.com/sns)
2. Click **Topics** → **Create topic**
3. Configure:
   - **Type:** Standard
   - **Name:** `dost-ptri-day4-[yourname]-signup-topic`
   - Leave everything else as default
4. Click **Create topic**
5. **Copy the Topic ARN** (e.g., `arn:aws:sns:ap-southeast-1:123456789012:dost-ptri-day4-...`)

## Step 2: Create Subscriber 1 — Email

1. In the topic page, click **Create subscription**
2. Configure:
   - **Protocol:** Email
   - **Endpoint:** Your real email address
3. Click **Create subscription**
4. **Check your email** → Click the **Confirm subscription** link

> ⚠️ You must confirm the email subscription or it won't receive messages.

## Step 3: Create Subscriber 2 — SQS Audit Queue

### Create the queue
1. Go to **SQS Console** → **Create queue**
2. Configure:
   - **Type:** Standard
   - **Name:** `dost-ptri-day4-[yourname]-signup-audit-queue`
3. Click **Create queue**

### Allow SNS to send messages to this queue
1. Select the queue → Click **Edit**
2. Scroll down to the **Access policy** section
3. Replace the policy with (update the ARNs):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {"Service": "sns.amazonaws.com"},
      "Action": "sqs:SendMessage",
      "Resource": "YOUR_SQS_QUEUE_ARN",
      "Condition": {
        "ArnEquals": {
          "aws:SourceArn": "YOUR_SNS_TOPIC_ARN"
        }
      }
    }
  ]
}
```

> Replace `YOUR_SQS_QUEUE_ARN` and `YOUR_SNS_TOPIC_ARN` with the actual ARNs.

3. Click **Save**

### Subscribe the queue to SNS
1. Go back to **SNS Console** → Your topic → **Create subscription**
2. Configure:
   - **Protocol:** Amazon SQS
   - **Endpoint:** Select `dost-ptri-day4-[yourname]-signup-audit-queue` ARN
3. Click **Create subscription**

## Step 4: Create Subscriber 3 — Lambda Welcome Processor

### Create the function
1. Go to **Lambda Console** → **Create function**
2. Configure:
   - **Function name:** `dost-ptri-day4-[yourname]-welcome-processor`
   - **Runtime:** Python 3.14
   - **Permissions:** Leave default
3. Click **Create function**
4. Replace `lambda_function.py` with:

```python
import json

def lambda_handler(event, context):
    for record in event["Records"]:
        sns_message = record["Sns"]
        subject = sns_message.get("Subject", "No Subject")
        body = json.loads(sns_message["Message"])

        username = body.get("username", "unknown")
        email = body.get("email", "unknown")
        plan = body.get("plan", "free")

        print(f"=== New Signup Received ===")
        print(f"  Username: {username}")
        print(f"  Email: {email}")
        print(f"  Plan: {plan}")
        print(f"  Action: Welcome email queued for {email}")
        print(f"  Action: Free trial activated for {plan} plan")

    return {"status": "processed", "count": len(event["Records"])}
```

5. Click **Deploy**

### Test the Lambda (before connecting to SNS)
1. Click **Test** → Configure test event:
   - **Event name:** `test-sns-event`
   - **Event JSON:**
   ```json
   {
     "Records": [
       {
         "Sns": {
           "Subject": "New Signup: maria.santos",
           "Message": "{\"username\": \"maria.santos\", \"email\": \"maria@example.com\", \"plan\": \"premium\"}"
         }
       }
     ]
   }
   ```
2. Click **Save**, then **Test**

**Expected output:**
```
=== New Signup Received ===
  Username: maria.santos
  Email: maria@example.com
  Plan: premium
  Action: Welcome email queued for maria@example.com
  Action: Free trial activated for premium plan
```

> ✅ The Lambda processes SNS events correctly. Now let's connect it to the real topic.

### Subscribe Lambda to SNS
1. In the Lambda function, click **Add trigger**
2. Configure:
   - **Source:** SNS
   - **SNS topic:** Select `dost-ptri-day4-[yourname]-signup-topic`
3. Click **Add**

## Step 5: Create the Publisher Lambda

1. Go to **Lambda Console** → **Create function**
2. Configure:
   - **Function name:** `dost-ptri-day4-[yourname]-signup-publisher`
   - **Runtime:** Python 3.14
   - **Permissions:** Leave default
3. Click **Create function**
4. Replace `lambda_function.py` with:

```python
import json
import boto3

sns = boto3.client("sns")

# REPLACE with your actual Topic ARN
TOPIC_ARN = "arn:aws:sns:YOUR_REGION:YOUR_ACCOUNT_ID:dost-ptri-day4-[yourname]-signup-topic"


def lambda_handler(event, context):
    signup = {
        "username": event.get("username", "newuser"),
        "email": event.get("email", "newuser@example.com"),
        "plan": event.get("plan", "free"),
        "source": "signup-form"
    }

    response = sns.publish(
        TopicArn=TOPIC_ARN,
        Subject=f"New Signup: {signup['username']}",
        Message=json.dumps(signup)
    )

    return {
        "status": "Signup published to SNS",
        "message_id": response["MessageId"],
        "signup": signup
    }
```

> ⚠️ **Replace the `TOPIC_ARN`** with your actual SNS topic ARN from Step 1.

5. Click **Deploy**

### Add SNS Permissions
1. Go to **Configuration** tab → **Permissions** → Click the **Role name**
2. Click **Add permissions** → **Attach policies**
3. Search and check ✅ `AmazonSNSFullAccess`
4. Click **Add permissions**

## Step 6: Test — Publish a Signup Event

1. Go to the **Publisher** Lambda (`dost-ptri-day4-[yourname]-signup-publisher`)
2. Click **Test** → Configure test event:
   - **Event name:** `test-signup`
   - **Event JSON:**
   ```json
   {
     "username": "maria.santos",
     "email": "maria@example.com",
     "plan": "premium"
   }
   ```
3. Click **Save**, then **Test**

**Expected output:**
```json
{
  "status": "Signup published to SNS",
  "message_id": "abc123...",
  "signup": {
    "username": "maria.santos",
    "email": "maria@example.com",
    "plan": "premium",
    "source": "signup-form"
  }
}
```

## Step 7: Verify All 3 Subscribers Received the Message

### ✅ Check 1: Email
- Check your inbox — you should receive an email with subject "New Signup: maria.santos" and the JSON body

### ✅ Check 2: SQS Audit Queue
1. Go to **SQS Console** → Select `dost-ptri-day4-[yourname]-signup-audit-queue`
2. Click **Send and receive messages** → **Poll for messages**
3. You should see a message — click it to view the body containing the signup data

### ✅ Check 3: Lambda Welcome Processor
1. Go to **Lambda Console** → `dost-ptri-day4-[yourname]-welcome-processor`
2. Click **Monitor** → **View CloudWatch logs**
3. You should see:

```
=== New Signup Received ===
  Username: maria.santos
  Email: maria@example.com
  Plan: premium
  Action: Welcome email queued for maria@example.com
  Action: Free trial activated for premium plan
```

## Step 8: Test Fan-Out with Multiple Signups

Send more test events from the Publisher:

| Event Name | username | plan |
|-----------|----------|------|
| `test-signup-2` | `juan.reyes` | `free` |
| `test-signup-3` | `ana.cruz` | `enterprise` |

Verify all 3 subscribers receive each message.

## What You Learned

- **SNS** implements the **pub/sub pattern** — one message, multiple subscribers
- **Fan-out** means a single publish delivers to email, SQS, and Lambda simultaneously
- SNS supports multiple subscriber types: Email, SQS, Lambda, HTTP, SMS
- **SQS as a subscriber** is useful for audit logs, buffering, and reliable processing
- **Lambda as a subscriber** enables real-time event processing
- This pattern is the foundation of **event-driven microservices** — services react to events without knowing about each other

## Cleanup

Delete in this order:

1. **SNS Subscriptions:** SNS Console → Topic → Delete all 3 subscriptions
2. **SNS Topic:** Delete `dost-ptri-day4-[yourname]-signup-topic`
3. **SQS Queue:** Delete `dost-ptri-day4-[yourname]-signup-audit-queue`
4. **Lambda (Welcome):** Delete `dost-ptri-day4-[yourname]-welcome-processor`
5. **Lambda (Publisher):** Delete `dost-ptri-day4-[yourname]-signup-publisher`

---

**← Previous:** [02 - Order Queue with SQS](./02-sqs-order-queue.md) | **Next:** [04 - React Integration App →](./04-react-integration-app.md) | **Back to:** [README](./README.md)
