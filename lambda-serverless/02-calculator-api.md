# Lab 2 — Calculator Lambda

## Goal
Create a Lambda function that performs basic math operations (add, subtract, multiply, divide) using direct invocation via the Lambda Console test feature.

## Services Used
- **AWS Lambda**

## Naming Convention
Replace `[yourname]` with your name (e.g., `juan`).

---

## Step 1: Create the Lambda Function

1. Go to **Lambda Console** → **Create function**
2. Select **Author from scratch**
3. Configure:
   - **Function name:** `dost-ptri-lambda-[yourname]-calculator`
   - **Runtime:** Python 3.12
   - **Architecture:** x86_64
   - **Permissions:** Leave default
4. Click **Create function**

## Step 2: Write the Code

Replace the contents of `lambda_function.py` with:

```python
import json

def lambda_handler(event, context):
    try:
        num1 = float(event["num1"])
        num2 = float(event["num2"])
        operation = event["operation"].lower()

        if operation == "add":
            result = num1 + num2
        elif operation == "subtract":
            result = num1 - num2
        elif operation == "multiply":
            result = num1 * num2
        elif operation == "divide":
            if num2 == 0:
                return {"error": "Cannot divide by zero"}
            result = num1 / num2
        else:
            return {"error": f"Unknown operation: {operation}. Use add, subtract, multiply, or divide."}

        return {
            "operation": operation,
            "num1": num1,
            "num2": num2,
            "result": result
        }

    except KeyError as e:
        return {"error": f"Missing required field: {str(e)}"}
    except Exception as e:
        return {"error": str(e)}
```

Click **Deploy**.

## Step 3: Test — Addition

1. Click **Test** → Configure test event:
   - **Event name:** `test-add`
   - **Event JSON:**
   ```json
   {
     "num1": 10,
     "num2": 5,
     "operation": "add"
   }
   ```
2. Click **Save**, then **Test**

**Expected output:**
```json
{
  "operation": "add",
  "num1": 10.0,
  "num2": 5.0,
  "result": 15.0
}
```

## Step 4: Test All Operations

Create a new test event for each (click **Test** dropdown → **Configure test event** → **Create new event**):

| Event Name | Event JSON | Expected Result |
|-----------|------------|----------------|
| `test-subtract` | `{"num1": 10, "num2": 3, "operation": "subtract"}` | `7.0` |
| `test-multiply` | `{"num1": 4, "num2": 5, "operation": "multiply"}` | `20.0` |
| `test-divide` | `{"num1": 20, "num2": 4, "operation": "divide"}` | `5.0` |
| `test-divide-zero` | `{"num1": 10, "num2": 0, "operation": "divide"}` | `"error": "Cannot divide by zero"` |
| `test-missing-field` | `{"num1": 10}` | `"error": "Missing required field: 'num2'"` |

> ✅ Run each test and verify the output matches.

## What You Learned

- Lambda can process structured JSON input and return results directly
- Error handling with `try/except` returns meaningful error messages
- You can create **multiple test events** to validate different scenarios
- No API Gateway needed — Lambda can be invoked directly for backend logic

## Cleanup

1. Lambda Console → Select `dost-ptri-lambda-[yourname]-calculator` → **Actions** → **Delete**

---

**← Previous:** [01 - Hello World](./01-hello-world.md) | **Next:** [03 - S3 Image Processor →](./03-s3-image-processor.md)
