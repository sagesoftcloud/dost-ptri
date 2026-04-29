# Lab 2 — Order Queue with SQS

## Goal
Build an asynchronous order processing system using Amazon SQS. A "producer" Lambda sends orders to an SQS queue, and a "consumer" Lambda automatically picks them up and processes them.

## Services Used
- **Amazon SQS** — Message queue
- **AWS Lambda** (×2) — Producer and consumer functions

## Naming Convention
Replace `[yourname]` with your name (e.g., `juan`).

---

## Step 1: Create the SQS Queue

1. Go to **SQS Console** → [https://console.aws.amazon.com/sqs](https://console.aws.amazon.com/sqs)
2. Click **Create queue**
3. Configure:
   - **Type:** Standard
   - **Name:** `dost-ptri-day4-[yourname]-order-queue`
4. Expand the **Configuration** section and set:
   - **Visibility timeout:** `30` seconds (default — leave as is)
   - **Message retention period:** `4` days (default — leave as is)
   - **Receive message wait time:** `5` seconds (enables long polling)
   - Leave everything else as default
5. Click **Create queue**
6. **Copy the Queue URL** (e.g., `https://sqs.ap-southeast-1.amazonaws.com/123456789012/dost-ptri-day4-...`)

## Step 2: Create the Producer Lambda (Send Orders)

1. Go to **Lambda Console** → **Create function**
2. Configure:
   - **Function name:** `dost-ptri-day4-[yourname]-order-producer`
   - **Runtime:** Python 3.14
   - **Permissions:** Leave default
3. Click **Create function**
4. Replace `lambda_function.py` with:

```python
import json
import boto3
from datetime import datetime, timezone

sqs = boto3.client("sqs")

# REPLACE with your actual Queue URL
QUEUE_URL = "https://sqs.YOUR_REGION.amazonaws.com/YOUR_ACCOUNT_ID/dost-ptri-day4-[yourname]-order-queue"


def lambda_handler(event, context):
    # Build the order from input
    order = {
        "order_id": event.get("order_id", "ORD-001"),
        "customer": event.get("customer", "Juan Dela Cruz"),
        "items": event.get("items", ["Item A", "Item B"]),
        "total": event.get("total", 150.00),
        "timestamp": datetime.now(timezone.utc).isoformat()
    }

    # Send to SQS
    response = sqs.send_message(
        QueueUrl=QUEUE_URL,
        MessageBody=json.dumps(order),
        MessageAttributes={
            "OrderType": {
                "DataType": "String",
                "StringValue": event.get("order_type", "standard")
            }
        }
    )

    return {
        "status": "Order sent to queue",
        "message_id": response["MessageId"],
        "order": order
    }
```

> ⚠️ **Replace the `QUEUE_URL`** with your actual SQS queue URL from Step 1.

5. Click **Deploy**

### Add SQS Permissions to the Producer
1. Go to **Configuration** tab → **Permissions**
2. Click the **Role name** link (opens IAM in a new tab)
3. Click **Add permissions** → **Attach policies**
4. Search and check ✅ `AmazonSQSFullAccess`
5. Click **Add permissions**

## Step 3: Create the Consumer Lambda (Process Orders)

1. Go to **Lambda Console** → **Create function**
2. Configure:
   - **Function name:** `dost-ptri-day4-[yourname]-order-consumer`
   - **Runtime:** Python 3.14
   - **Permissions:** Leave default
3. Click **Create function**
4. Replace `lambda_function.py` with:

```python
import json

def lambda_handler(event, context):
    print(f"Received {len(event['Records'])} message(s)")

    for record in event["Records"]:
        body = json.loads(record["body"])

        order_id = body.get("order_id", "unknown")
        customer = body.get("customer", "unknown")
        total = body.get("total", 0)
        items = body.get("items", [])

        print(f"Processing Order: {order_id}")
        print(f"  Customer: {customer}")
        print(f"  Items: {', '.join(items)}")
        print(f"  Total: ${total:.2f}")
        print(f"  Status: ✅ PROCESSED")

    return {"status": "processed", "count": len(event["Records"])}
```

5. Click **Deploy**

### Test the Consumer Lambda (before connecting to SQS)
1. Click **Test** → Configure test event:
   - **Event name:** `test-sqs-event`
   - **Event JSON:**
   ```json
   {
     "Records": [
       {
         "body": "{\"order_id\": \"ORD-TEST\", \"customer\": \"Test User\", \"items\": [\"Laptop\", \"Mouse\"], \"total\": 46350.00}"
       }
     ]
   }
   ```
2. Click **Save**, then **Test**

**Expected output (in Execution results):**
```
Received 1 message(s)
Processing Order: ORD-TEST
  Customer: Test User
  Items: Laptop, Mouse
  Total: $46350.00
  Status: ✅ PROCESSED
```

> ✅ The consumer works! Now let's connect it to the real SQS queue.

## Step 4: Connect SQS to the Consumer Lambda

1. In the `dost-ptri-day4-[yourname]-order-consumer` function, click **Add trigger**
2. Configure:
   - **Source:** SQS
   - **SQS queue:** Select `dost-ptri-day4-[yourname]-order-queue`
   - **Batch size:** `5`
   - **Batch window:** `0`
3. Click **Add**

> If you get a permissions error, add `AmazonSQSFullAccess` to the consumer's role (same as Step 2).

## Step 5: Test — Send an Order

1. Go to the **Producer** Lambda (`dost-ptri-day4-[yourname]-order-producer`)
2. Click **Test** → Configure test event:
   - **Event name:** `test-order-1`
   - **Event JSON:**
   ```json
   {
     "order_id": "ORD-101",
     "customer": "Maria Santos",
     "items": ["Laptop", "Mouse", "Keyboard"],
     "total": 45500.00,
     "order_type": "express"
   }
   ```
3. Click **Save**, then **Test**

**Expected output:**
```json
{
  "status": "Order sent to queue",
  "message_id": "abc123...",
  "order": {
    "order_id": "ORD-101",
    "customer": "Maria Santos",
    "items": ["Laptop", "Mouse", "Keyboard"],
    "total": 45500.0,
    "timestamp": "2026-04-28T..."
  }
}
```

## Step 6: Verify the Consumer Processed It

1. Go to the **Consumer** Lambda (`dost-ptri-day4-[yourname]-order-consumer`)
2. Click **Monitor** tab → **View CloudWatch logs**
3. Click the latest log stream
4. You should see:

```
Received 1 message(s)
Processing Order: ORD-101
  Customer: Maria Santos
  Items: Laptop, Mouse, Keyboard
  Total: $45500.00
  Status: ✅ PROCESSED
```

## Step 7: Send Multiple Orders

Create more test events on the Producer and fire them:

| Event Name | order_id | customer | total |
|-----------|----------|----------|-------|
| `test-order-2` | `ORD-102` | `Juan Reyes` | `1200.00` |
| `test-order-3` | `ORD-103` | `Ana Cruz` | `8900.50` |

Check the Consumer's CloudWatch logs — each order should appear as processed.

## Step 8: Check the SQS Queue Metrics

1. Go to **SQS Console** → Select your queue
2. Click **Monitoring** tab
3. You should see:
   - **Messages Sent** — matches the number of orders you sent
   - **Messages Received** — matches (consumer picked them up)
   - **Messages in Queue** — should be `0` (all processed)

## What You Learned

- **SQS** decouples the producer from the consumer — they don't need to run at the same time
- The producer sends messages to the queue; the consumer processes them asynchronously
- **SQS triggers** on Lambda automatically invoke the consumer when messages arrive
- If the consumer fails, the message returns to the queue after the visibility timeout
- This pattern is the foundation of **event-driven architectures** and **microservices**

## Cleanup

> ⚠️ **If you plan to continue to Labs 3 and 4 — do NOT delete the SQS queue yet!** Lab 4 reuses it.

1. **SQS Queue:** SQS Console → Select `dost-ptri-day4-[yourname]-order-queue` → **Delete**
2. **Producer Lambda:** Delete `dost-ptri-day4-[yourname]-order-producer`
3. **Consumer Lambda:** Delete `dost-ptri-day4-[yourname]-order-consumer`

---

**← Previous:** [01 - REST API with Cognito Auth](./01-api-gateway-cognito-auth.md) | **Next:** [03 - SNS Fan-Out Notifications →](./03-sns-fanout-notifications.md)
