# Lab 1 — Hello World Lambda

## Goal
Create your first Lambda function that returns a greeting message. This is the simplest possible Lambda to understand the basics.

## Services Used
- **AWS Lambda**

## Naming Convention
Replace `[yourname]` with your name (e.g., `juan`).

---

## Step 1: Create the Lambda Function

1. Go to **Lambda Console** → [https://console.aws.amazon.com/lambda](https://console.aws.amazon.com/lambda)
2. Click **Create function**
3. Select **Author from scratch**
4. Configure:
   - **Function name:** `dost-ptri-lambda-[yourname]-hello-world`
   - **Runtime:** Python 3.12
   - **Architecture:** x86_64
   - **Permissions:** Leave default (Create a new role with basic Lambda permissions)
5. Click **Create function**

## Step 2: Write the Code

1. In the **Code source** section, you'll see the inline code editor
2. Replace the contents of `lambda_function.py` with:

```python
import json

def lambda_handler(event, context):
    # Get the name from the event, default to "World"
    name = event.get("name", "World")

    message = f"Hello, {name}! Welcome to AWS Lambda - DOST PTRI Training!"

    return {
        "statusCode": 200,
        "body": json.dumps({
            "message": message,
            "input_received": event
        })
    }
```

3. Click **Deploy**

> ✅ You should see "Successfully updated the function"

## Step 3: Test the Function

### Test with default event
1. Click **Test**
2. Configure test event:
   - **Event name:** `test-default`
   - **Event JSON:**
   ```json
   {}
   ```
3. Click **Save**, then click **Test**

**Expected output:**
```json
{
  "statusCode": 200,
  "body": "{\"message\": \"Hello, World! Welcome to AWS Lambda - DOST PTRI Training!\", \"input_received\": {}}"
}
```

### Test with a name
1. Click **Test** → select the dropdown → **Configure test event**
2. Create a new event:
   - **Event name:** `test-with-name`
   - **Event JSON:**
   ```json
   {
     "name": "Juan"
   }
   ```
3. Click **Save**, then click **Test**

**Expected output:**
```json
{
  "statusCode": 200,
  "body": "{\"message\": \"Hello, Juan! Welcome to AWS Lambda - DOST PTRI Training!\", \"input_received\": {\"name\": \"Juan\"}}"
}
```

## Step 4: View Logs

1. Click the **Monitor** tab
2. Click **View CloudWatch logs**
3. Click on the latest log stream
4. You should see your function's execution logs (START, END, REPORT)

> 💡 The REPORT line shows: **Duration** (how long it ran), **Billed Duration**, and **Memory Used**

## What You Learned

- How to create a Lambda function from scratch
- Lambda receives input via the `event` parameter
- Lambda returns a response as a dictionary
- CloudWatch Logs automatically captures Lambda execution logs

## Cleanup

1. Go to **Lambda Console** → **Functions**
2. Select `dost-ptri-lambda-[yourname]-hello-world`
3. Click **Actions** → **Delete** → Confirm

---

**Next:** [02 - Calculator API →](./02-calculator-api.md)
