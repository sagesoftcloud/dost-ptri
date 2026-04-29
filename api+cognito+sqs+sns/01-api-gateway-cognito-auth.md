# Lab 1 — REST API with Cognito Authentication

## Goal
Create a REST API using API Gateway, backed by a Lambda function, and protect it with Amazon Cognito so only authenticated users can access it.

## Services Used
- **Amazon API Gateway** — REST API endpoint
- **AWS Lambda** — Backend logic
- **Amazon Cognito** — User pool for authentication

## Naming Convention
Replace `[yourname]` with your name (e.g., `juan`).

---

## Step 1: Create the Lambda Backend

1. Go to **Lambda Console** → **Create function**
2. Configure:
   - **Function name:** `dost-ptri-day4-[yourname]-api-backend`
   - **Runtime:** Python 3.14
   - **Permissions:** Leave default
3. Click **Create function**
4. Replace `lambda_function.py` with:

```python
import json

def lambda_handler(event, context):
    # Get the authenticated user info from Cognito (via API Gateway)
    claims = event.get("requestContext", {}).get("authorizer", {}).get("claims", {})
    username = claims.get("cognito:username", "anonymous")
    email = claims.get("email", "unknown")

    http_method = event.get("httpMethod", "GET")
    path = event.get("path", "/")

    return {
        "statusCode": 200,
        "headers": {"Content-Type": "application/json"},
        "body": json.dumps({
            "message": f"Hello {username}! You are authenticated.",
            "email": email,
            "method": http_method,
            "path": path,
            "data": [
                {"id": 1, "name": "Sample Item 1"},
                {"id": 2, "name": "Sample Item 2"},
                {"id": 3, "name": "Sample Item 3"}
            ]
        })
    }
```

5. Click **Deploy**

### Test the Lambda
1. Click **Test** → Configure test event
   - **Event name:** `test-api-event`
   - **Event JSON:**
   ```json
   {
     "httpMethod": "GET",
     "path": "/items",
     "requestContext": {
       "authorizer": {
         "claims": {
           "cognito:username": "juan@example.com",
           "email": "juan@example.com"
         }
       }
     }
   }
   ```
2. Click **Save**, then **Test**

**Expected output:**
```json
{
  "statusCode": 200,
  "headers": {"Content-Type": "application/json"},
  "body": "{\"message\": \"Hello juan@example.com! You are authenticated.\", \"email\": \"juan@example.com\", \"method\": \"GET\", \"path\": \"/items\", \"data\": [{\"id\": 1, \"name\": \"Sample Item 1\"}, {\"id\": 2, \"name\": \"Sample Item 2\"}, {\"id\": 3, \"name\": \"Sample Item 3\"}]}"
}
```

> ✅ The Lambda works! The test event simulates what API Gateway will send after Cognito validates a token.

## Step 2: Create the Cognito User Pool

1. Go to **Cognito Console** → [https://console.aws.amazon.com/cognito](https://console.aws.amazon.com/cognito)
2. Click **Create user pool**

### Define your application
1. **Application type:** Select **Single-page application (SPA)**
   > This creates a public client with no client secret — correct for our React app.
2. **Name your application:** `dost-ptri-day4-[yourname]-app`

### Configure options
1. **Options for sign-in identifiers:** Check ✅ **Email**
2. **Required attributes for sign-up:** `email` (should already be selected)

### Add a return URL
1. **Return URL:** `http://localhost:3000`
   > This is where Cognito redirects after login. We'll use this in Lab 4 (React app).

### Create
1. Click **Create your application**
2. You'll see the **Set up your application** page with code examples — scroll down and click **Go to overview**

> ℹ️ The quick-create flow automatically creates a **Cognito domain** and configures **managed login** for you. No need to set these up manually.

### Note the IDs
1. You're now in your user pool overview
2. Note the **User pool ID** (e.g., `ap-southeast-1_AbCdEfGhI`) — shown at the top
3. In the left sidebar, click **App clients** → click your app client → note the **Client ID**

### Note the Cognito Domain
1. In the left sidebar, click **Domain**
2. Note the full domain URL (e.g., `https://abc123def.auth.ap-southeast-1.amazoncognito.com`)
   > This was auto-created. You'll need it for Lab 4 (React login).

### Configure OAuth settings for the React app

The quick-create sets up **Authorization code grant** by default. We also need **Implicit grant** for simpler browser-based token exchange:

1. In the left sidebar, click **App clients** → click your app client
2. Click the **Login pages** tab
3. Click **Edit**
4. Under **OAuth 2.0 grant types:**
   - ✅ Keep **Authorization code grant** checked
   - ✅ Also check **Implicit grant**
5. Under **OpenID Connect scopes**, verify these are checked:
   - ✅ `openid`, ✅ `email`, ✅ `profile`
6. Under **Allowed callback URLs**, verify `http://localhost:3000` is listed
7. Under **Allowed sign-out URLs**, add: `http://localhost:3000`
8. Click **Save changes**

### Enable USER_PASSWORD_AUTH (for CLI testing)

The SPA client only enables SRP auth by default. We need password auth for the CLI test later:

1. Still in your app client, click the **App client information** tab
2. Scroll to **Authentication flows** → click **Edit**
3. Check ✅ **ALLOW_USER_PASSWORD_AUTH**
4. Click **Save changes**

### Disable MFA (for lab simplicity)

1. In the left sidebar, click **Sign-in** (under **Authentication**)
2. Under **Multi-factor authentication**, click **Edit**
3. Select **No MFA**
4. Click **Save changes**

## Step 3: Create a Test User

1. In the left sidebar, click **Users** (under **User management**)
2. Click **Create user**
3. Configure:
   - **Invitation message:** Send email invitation
   - **Email address:** your real email (e.g., `juan@example.com`)
   - **Temporary password:** Set a password → `TempPass123!`
4. Click **Create user**

> ℹ️ Since we chose **Email** as the sign-in identifier, the email address acts as the username. There is no separate username field.

## Step 4: Create the API Gateway

1. Go to **API Gateway Console** → **Create API**
2. Under **REST API**, click **Build**
3. Configure:
   - **API name:** `dost-ptri-day4-[yourname]-api`
   - **API endpoint type:** Regional
   - **IP address type:** IPv4
4. Click **Create API**

## Step 5: Create the Cognito Authorizer

1. In your API, click **Authorizers** (left sidebar)
2. Click **Create authorizer**
3. Configure:
   - **Authorizer name:** `dost-ptri-day4-[yourname]-cognito-auth`
   - **Authorizer type:** Cognito
   - **Cognito user pool:** Select your user pool (the one you just created)
   - **Token source:** `Authorization`
4. Click **Create authorizer**

## Step 6: Create Resource and Method

### Create the resource
1. Click **Resources** (left sidebar)
2. Click **Create resource**
   - **Resource name:** `items`
   - **Resource path:** `/items`
3. Click **Create resource**

### Create the GET method
1. Select `/items` → Click **Create method**
2. Configure:
   - **Method type:** GET
   - **Integration type:** Lambda Function
   - **Lambda proxy integration:** ✅ ON
   - **Lambda function:** `dost-ptri-day4-[yourname]-api-backend`
3. Click **Create method**

### Attach the Cognito Authorizer
1. Select the **GET** method under `/items`
2. Click the **Method request** tab → **Edit**
3. Set:
   - **Authorization:** Select `dost-ptri-day4-[yourname]-cognito-auth`
4. Click **Save**

## Step 7: Deploy the API

1. Click **Deploy API**
2. **Stage:** New stage → **Stage name:** `prod`
3. Click **Deploy**
4. Copy the **Invoke URL** (e.g., `https://abc123.execute-api.ap-southeast-1.amazonaws.com/prod`)

## Step 8: Test — Without Authentication (Should Fail)

Open a browser or use curl:

```bash
curl https://YOUR_API_URL/prod/items
```

**Expected response:**
```json
{"message": "Unauthorized"}
```

> ✅ This proves the Cognito authorizer is blocking unauthenticated requests.

## Step 9: Get an Authentication Token

To get a token, we'll use the AWS CLI. Open **CloudShell** from the AWS Console top bar.

### Initiate auth (first login with temp password)
```bash
aws cognito-idp initiate-auth \
  --auth-flow USER_PASSWORD_AUTH \
  --client-id YOUR_CLIENT_ID \
  --auth-parameters USERNAME=your@email.com,PASSWORD=TempPass123! \
  --region YOUR_REGION
```

> If you get a `NEW_PASSWORD_REQUIRED` challenge, copy the `Session` value from the output and run:

```bash
aws cognito-idp respond-to-auth-challenge \
  --client-id YOUR_CLIENT_ID \
  --challenge-name NEW_PASSWORD_REQUIRED \
  --session "PASTE_THE_LONG_SESSION_STRING_HERE" \
  --challenge-responses USERNAME=your@email.com,NEW_PASSWORD=MyNewPass123! \
  --region YOUR_REGION
```

> ⚠️ Use `NEW_PASSWORD=` (not `PASSWORD=`). The session expires quickly — run both commands back to back.

### Copy the IdToken
From the response, copy the `IdToken` value (it's a long JWT string).

## Step 10: Test — With Authentication (Should Succeed)

```bash
curl -H "Authorization: YOUR_ID_TOKEN" https://YOUR_API_URL/prod/items
```

**Expected response:**
```json
{
  "message": "Hello your@email.com! You are authenticated.",
  "email": "your@email.com",
  "method": "GET",
  "path": "/items",
  "data": [
    {"id": 1, "name": "Sample Item 1"},
    {"id": 2, "name": "Sample Item 2"},
    {"id": 3, "name": "Sample Item 3"}
  ]
}
```

> ✅ The API returns data only when a valid Cognito token is provided.

## What You Learned

- **API Gateway** creates REST API endpoints accessible over HTTP
- **Cognito User Pools** manage user registration, login, and token generation
- **Cognito Authorizer** on API Gateway validates JWT tokens before allowing access
- Without a valid token, the API returns `401 Unauthorized`
- The Lambda function receives the authenticated user's claims (username, email) via the event

## Cleanup

> ⚠️ **If you plan to continue to Labs 2, 3, and 4 — do NOT delete these resources yet!** Lab 4 reuses the Cognito User Pool and API Gateway.

1. **API Gateway:** Delete `dost-ptri-day4-[yourname]-api`
2. **Lambda:** Delete `dost-ptri-day4-[yourname]-api-backend`
3. **Cognito:** Cognito Console → Select your user pool → **Delete user pool**

---

**Next:** [02 - Order Queue with SQS →](./02-sqs-order-queue.md)
