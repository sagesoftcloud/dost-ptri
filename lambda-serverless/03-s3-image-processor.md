# Lab 3 — S3 Event Trigger: Image Processor

## Goal
Create a Lambda function that automatically triggers when an image is uploaded to an S3 bucket. The function reads the image metadata (size, type, dimensions) and saves a summary to a separate output bucket.

## Services Used
- **AWS Lambda**
- **Amazon S3** (2 buckets: input + output)
- **AWS IAM** (Lambda execution role with S3 permissions)
- **Amazon CloudWatch** (logs)

## Naming Convention
Replace `[yourname]` with your name (e.g., `juan`).

---

## Step 1: Create the S3 Buckets

### Input bucket (where you upload images)
1. Go to **S3 Console** → [https://console.aws.amazon.com/s3](https://console.aws.amazon.com/s3)
2. Click **Create bucket**
3. Configure:
   - **Bucket name:** `dost-ptri-lambda-[yourname]-images-input`
   - **Region:** Your chosen region
   - **Block all public access:** ✅ Keep enabled (default)
   - Leave everything else as default
4. Click **Create bucket**

### Output bucket (where Lambda writes results)
1. Click **Create bucket**
2. Configure:
   - **Bucket name:** `dost-ptri-lambda-[yourname]-images-output`
   - **Region:** Same region as input bucket
   - **Block all public access:** ✅ Keep enabled
3. Click **Create bucket**

> ⚠️ S3 bucket names must be globally unique. If the name is taken, add a number (e.g., `-01`).

## Step 2: Create the IAM Role for Lambda

Lambda needs permission to read from the input bucket and write to the output bucket.

1. Go to **IAM Console** → [https://console.aws.amazon.com/iam](https://console.aws.amazon.com/iam)
2. Click **Roles** → **Create role**
3. Configure:
   - **Trusted entity type:** AWS service
   - **Use case:** Lambda
4. Click **Next**
5. Attach these policies — search and check ✅ each:
   - `AmazonS3ReadOnlyAccess`
   - `AmazonS3FullAccess`
   - `CloudWatchLogsFullAccess`

> 💡 In production, you'd create a custom policy with least-privilege access. For this lab, these managed policies are fine.

6. Click **Next**
7. Configure:
   - **Role name:** `dost-ptri-lambda-[yourname]-image-processor-role`
   - **Description:** `Lambda role for S3 image processor lab`
8. Click **Create role**

## Step 3: Create the Lambda Function

1. Go to **Lambda Console** → **Create function**
2. Select **Author from scratch**
3. Configure:
   - **Function name:** `dost-ptri-lambda-[yourname]-image-processor`
   - **Runtime:** Python 3.12
   - **Architecture:** x86_64
   - **Permissions:** Expand **Change default execution role**
     - Select **Use an existing role**
     - Choose `dost-ptri-lambda-[yourname]-image-processor-role`
4. Click **Create function**

## Step 4: Configure the Function Settings

1. Go to the **Configuration** tab → **General configuration** → **Edit**
2. Set:
   - **Timeout:** `30` seconds (images may take longer to process)
   - **Memory:** `256` MB
3. Click **Save**

## Step 5: Write the Code

Replace the contents of `lambda_function.py` with:

```python
import json
import boto3
import os
from datetime import datetime, timezone
from urllib.parse import unquote_plus

s3_client = boto3.client("s3")

# Output bucket name — REPLACE [yourname] with your actual name
OUTPUT_BUCKET = "dost-ptri-lambda-[yourname]-images-output"


def lambda_handler(event, context):
    print("Received event:", json.dumps(event))

    for record in event["Records"]:
        # Get bucket and file info from the S3 event
        source_bucket = record["s3"]["bucket"]["name"]
        source_key = unquote_plus(record["s3"]["object"]["key"])
        file_size = record["s3"]["object"]["size"]
        event_time = record["eventTime"]

        print(f"Processing: s3://{source_bucket}/{source_key}")

        # Get the object metadata from S3
        try:
            response = s3_client.head_object(
                Bucket=source_bucket,
                Key=source_key
            )
        except Exception as e:
            print(f"Error reading object: {e}")
            continue

        content_type = response.get("ContentType", "unknown")
        last_modified = str(response.get("LastModified", "unknown"))

        # Determine file extension
        file_extension = os.path.splitext(source_key)[1].lower()
        file_name = os.path.basename(source_key)

        # Build the summary
        summary = {
            "file_name": file_name,
            "source_bucket": source_bucket,
            "source_key": source_key,
            "file_size_bytes": file_size,
            "file_size_readable": format_size(file_size),
            "content_type": content_type,
            "file_extension": file_extension,
            "last_modified": last_modified,
            "event_time": event_time,
            "processed_at": datetime.now(timezone.utc).isoformat(),
            "is_image": file_extension in [".jpg", ".jpeg", ".png", ".gif", ".bmp", ".webp"],
            "status": "processed"
        }

        print(f"Summary: {json.dumps(summary, indent=2)}")

        # Save summary to output bucket
        output_key = f"processed/{file_name}.json"
        s3_client.put_object(
            Bucket=OUTPUT_BUCKET,
            Key=output_key,
            Body=json.dumps(summary, indent=2),
            ContentType="application/json"
        )

        print(f"Summary saved to s3://{OUTPUT_BUCKET}/{output_key}")

    return {
        "statusCode": 200,
        "body": json.dumps({"message": "Processing complete"})
    }


def format_size(size_bytes):
    """Convert bytes to human-readable format."""
    if size_bytes < 1024:
        return f"{size_bytes} B"
    elif size_bytes < 1024 * 1024:
        return f"{size_bytes / 1024:.1f} KB"
    else:
        return f"{size_bytes / (1024 * 1024):.1f} MB"
```

> ⚠️ **Important:** Replace `[yourname]` in the `OUTPUT_BUCKET` variable with your actual name.

Click **Deploy**.

## Step 6: Add the S3 Trigger

1. In the Lambda function page, click **Add trigger**
2. Configure:
   - **Source:** S3
   - **Bucket:** Select `dost-ptri-lambda-[yourname]-images-input`
   - **Event types:** ✅ **All object create events** (`s3:ObjectCreated:*`)
   - **Prefix:** (leave empty)
   - **Suffix:** (leave empty)
   - Check ✅ **I acknowledge that using the same S3 bucket for both input and output is not recommended...**
3. Click **Add**

> ✅ You should see the S3 trigger appear in the function's **Designer** diagram at the top.

## Step 7: Test — Upload an Image

1. Go to **S3 Console** → Select `dost-ptri-lambda-[yourname]-images-input`
2. Click **Upload**
3. Click **Add files** → Select any image file from your computer (JPG, PNG, etc.)
4. Click **Upload**

## Step 8: Verify the Results

### Check the output bucket
1. Go to **S3 Console** → Select `dost-ptri-lambda-[yourname]-images-output`
2. Open the `processed/` folder
3. You should see a `.json` file with the same name as your uploaded image
4. Click on it → **Download** or **Open** to view the summary

**Example output (your-image.jpg.json):**
```json
{
  "file_name": "your-image.jpg",
  "source_bucket": "dost-ptri-lambda-juan-images-input",
  "source_key": "your-image.jpg",
  "file_size_bytes": 245678,
  "file_size_readable": "239.9 KB",
  "content_type": "image/jpeg",
  "file_extension": ".jpg",
  "last_modified": "2026-04-28 10:00:00+00:00",
  "event_time": "2026-04-28T10:00:00.000Z",
  "processed_at": "2026-04-28T10:00:01.123456+00:00",
  "is_image": true,
  "status": "processed"
}
```

### Check CloudWatch Logs
1. Go to **Lambda Console** → Select your function → **Monitor** tab
2. Click **View CloudWatch logs**
3. Click the latest log stream
4. You should see:
   - `Received event:` with the full S3 event
   - `Processing: s3://...`
   - `Summary:` with the file details
   - `Summary saved to s3://...`

## Step 9: Test with Multiple Files

Upload 2-3 more files (different types and sizes) to the input bucket. Check that each one generates a corresponding `.json` summary in the output bucket.

| Upload | Expected Output |
|--------|----------------|
| `photo.png` | `processed/photo.png.json` with `is_image: true` |
| `document.pdf` | `processed/document.pdf.json` with `is_image: false` |
| `screenshot.jpg` | `processed/screenshot.jpg.json` with `is_image: true` |

## What You Learned

- Lambda can be **triggered automatically** by S3 events (no manual invocation needed)
- The S3 event provides bucket name, object key, and file size in the `event` parameter
- Lambda can **read from one S3 bucket** and **write to another**
- IAM roles control what AWS services Lambda can access
- CloudWatch Logs captures all `print()` output from Lambda
- This pattern (S3 upload → Lambda processing → output) is the foundation of serverless data pipelines

## Cleanup

Delete in this order:

1. **S3 input bucket:** S3 Console → Select `dost-ptri-lambda-[yourname]-images-input` → **Empty** → then **Delete**
2. **S3 output bucket:** S3 Console → Select `dost-ptri-lambda-[yourname]-images-output` → **Empty** → then **Delete**
3. **Lambda function:** Lambda Console → Select `dost-ptri-lambda-[yourname]-image-processor` → **Actions** → **Delete**
4. **IAM role:** IAM Console → Roles → Search `dost-ptri-lambda-[yourname]-image-processor-role` → **Delete**
5. **CloudWatch log group:** CloudWatch Console → Log groups → Delete `/aws/lambda/dost-ptri-lambda-[yourname]-image-processor`

> ⚠️ You must **empty** S3 buckets before you can delete them.

---

**← Previous:** [02 - Calculator API](./02-calculator-api.md) | **Back to:** [README](./README.md)
