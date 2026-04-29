# Lab 4 — React Integration App (Putting It All Together)

## Goal
Build a simple React app running locally that integrates the services from Labs 1–3: **Cognito login**, **API Gateway calls**, **SQS order submission**, and **SNS notifications** — all from the browser.

## Services Used
- **Amazon Cognito** — Login/signup (from Lab 1)
- **API Gateway + Lambda** — Backend API (from Lab 1, extended)
- **Amazon SQS** — Order queue (from Lab 2)
- **Amazon SNS** — Notification fan-out (from Lab 3)

## Prerequisites
- Completed Labs 1, 2, and 3 (resources still running)
- **Node.js 18+** installed on your laptop (see installation guide below)
- A code editor (VS Code recommended)

### Installing Node.js

#### macOS
Option 1 — Download the installer:
1. Go to [https://nodejs.org](https://nodejs.org)
2. Download the **LTS** version (`.pkg` file)
3. Double-click the `.pkg` file and follow the installer

Option 2 — Using Homebrew (if installed):
```bash
brew install node
```

#### Windows
1. Go to [https://nodejs.org](https://nodejs.org)
2. Download the **LTS** version (`.msi` file)
3. Run the installer — check ✅ **Automatically install the necessary tools** when prompted
4. Click through the defaults → **Install** → **Finish**
5. Open **Command Prompt** or **PowerShell** and verify:

```bash
node --version
npm --version
```

> ℹ️ Both commands should return version numbers (e.g., `v22.x.x` and `10.x.x`). If you get "command not found", restart your terminal.

---

## Part A — Prepare the AWS Backend

Before building the React app, we need to extend the API Gateway from Lab 1 to handle orders.

### Step A1: Create the Order API Lambda

This single Lambda handles placing orders (sends to SQS + publishes to SNS) and listing items.

1. Go to **Lambda Console** → **Create function**
2. Configure:
   - **Function name:** `dost-ptri-day4-[yourname]-app-backend`
   - **Runtime:** Python 3.14
3. Click **Create function**
4. Replace `lambda_function.py` with:

```python
import json
import boto3
from datetime import datetime, timezone

sqs = boto3.client("sqs")
sns = boto3.client("sns")

# REPLACE these with your actual values from Labs 2 and 3
QUEUE_URL = "https://sqs.YOUR_REGION.amazonaws.com/YOUR_ACCOUNT_ID/dost-ptri-day4-[yourname]-order-queue"
TOPIC_ARN = "arn:aws:sns:YOUR_REGION:YOUR_ACCOUNT_ID:dost-ptri-day4-[yourname]-signup-topic"

PRODUCTS = [
    {"id": 1, "name": "Laptop", "price": 45500.00},
    {"id": 2, "name": "Mouse", "price": 850.00},
    {"id": 3, "name": "Keyboard", "price": 1200.00},
    {"id": 4, "name": "Monitor", "price": 12800.00},
    {"id": 5, "name": "Webcam", "price": 2500.00},
]


def lambda_handler(event, context):
    method = event.get("httpMethod", "GET")
    path = event.get("path", "/")

    # Get authenticated user from Cognito
    claims = event.get("requestContext", {}).get("authorizer", {}).get("claims", {})
    username = claims.get("cognito:username", "anonymous")
    email = claims.get("email", "unknown")

    headers = {
        "Content-Type": "application/json",
        "Access-Control-Allow-Origin": "*",
        "Access-Control-Allow-Headers": "Content-Type,Authorization",
        "Access-Control-Allow-Methods": "GET,POST,OPTIONS"
    }

    # GET /items — list products
    if method == "GET" and "/items" in path:
        return {
            "statusCode": 200,
            "headers": headers,
            "body": json.dumps({"user": username, "products": PRODUCTS})
        }

    # POST /orders — place an order
    if method == "POST" and "/orders" in path:
        body = json.loads(event.get("body", "{}"))
        items = body.get("items", [])
        total = body.get("total", 0)

        order = {
            "order_id": f"ORD-{datetime.now(timezone.utc).strftime('%H%M%S')}",
            "customer": username,
            "email": email,
            "items": items,
            "total": total,
            "timestamp": datetime.now(timezone.utc).isoformat()
        }

        # Send to SQS (Lab 2)
        sqs.send_message(
            QueueUrl=QUEUE_URL,
            MessageBody=json.dumps(order)
        )

        # Publish to SNS (Lab 3)
        sns.publish(
            TopicArn=TOPIC_ARN,
            Message=json.dumps(order),
            Subject="New Order Placed"
        )

        return {
            "statusCode": 200,
            "headers": headers,
            "body": json.dumps({"message": "Order placed!", "order": order})
        }

    return {
        "statusCode": 404,
        "headers": headers,
        "body": json.dumps({"message": "Not found"})
    }
```

> ⚠️ **Replace `QUEUE_URL` and `TOPIC_ARN`** with your actual values from Labs 2 and 3.
> - `QUEUE_URL` is the **URL** (starts with `https://sqs.`), NOT the ARN. Find it in SQS Console → select your queue → URL is at the top.
> - `TOPIC_ARN` is the **ARN** (starts with `arn:aws:sns:`). Find it in SNS Console → Topics → click your topic.

5. Click **Deploy**

### Test the Lambda — GET /items
1. Click **Test** → Configure test event:
   - **Event name:** `test-get-items`
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

**Expected:** `statusCode: 200` with a list of 5 products.

### Test the Lambda — POST /orders
1. Click **Test** → Create a new test event:
   - **Event name:** `test-post-order`
   - **Event JSON:**
   ```json
   {
     "httpMethod": "POST",
     "path": "/orders",
     "requestContext": {
       "authorizer": {
         "claims": {
           "cognito:username": "juan@example.com",
           "email": "juan@example.com"
         }
       }
     },
     "body": "{\"items\": [\"Laptop\", \"Mouse\"], \"total\": 46350.00}"
   }
   ```
2. Click **Save**, then **Test**

**Expected:** `statusCode: 200` with `"message": "Order placed!"` and the order details.

> ⚠️ The POST /orders test will fail if you haven't replaced `QUEUE_URL` and `TOPIC_ARN` with real values, or if the Lambda role doesn't have SQS/SNS permissions yet. You can test GET /items first, then test POST /orders after adding permissions below.

### Add Permissions to the Lambda Role
1. Go to **Configuration** → **Permissions** → Click the **Role name**
2. Attach these policies:
   - ✅ `AmazonSQSFullAccess`
   - ✅ `AmazonSNSFullAccess`

### Step A2: Add the `/orders` Route to API Gateway

1. Go to **API Gateway Console** → Select `dost-ptri-day4-[yourname]-api`

#### Create `/orders` resource
1. Click **Resources** → Select `/` → **Create resource**
   - **Resource name:** `orders`
2. Click **Create resource**

#### Create POST method on `/orders`
1. Select `/orders` → **Create method**
   - **Method type:** POST
   - **Integration type:** Lambda Function
   - **Lambda proxy integration:** ✅ ON
   - **Lambda function:** `dost-ptri-day4-[yourname]-app-backend`
2. Click **Create method**
3. Select the **POST** method → **Method request** → **Edit**
   - **Authorization:** Select your Cognito authorizer
4. Click **Save**

> ℹ️ The `/items` GET route from Lab 1 stays as is — it already works with the Lab 1 Lambda.

#### Update `/items` GET to use the new Lambda
The Lab 4 Lambda has the product catalog and CORS headers. We need to switch `/items` to use it:

1. Select `/items` → Select the **GET** method
2. Click **Integration request** → **Edit**
3. Change **Lambda function** to `dost-ptri-day4-[yourname]-app-backend`
4. Make sure **Lambda proxy integration** is ✅ ON
5. Click **Save**

### Step A3: Enable CORS

For each resource (`/items` and `/orders`):

1. Select the resource → **Enable CORS**
2. Configure:
   - **Access-Control-Allow-Origin:** `*`
   - **Access-Control-Allow-Headers:** `Content-Type,Authorization`
   - **Access-Control-Allow-Methods:** Check ✅ GET, POST, OPTIONS
3. Click **Save**

### Step A4: Re-deploy the API

> This deploys the **entire API** (all routes including `/items` and `/orders`) to the `prod` stage.

1. Click **Deploy API** (button at the top of the Resources page)
2. **Stage:** Select `prod`
3. Click **Deploy**
4. Your **Invoke URL** is the same as Lab 1 (e.g., `https://abc123.execute-api.us-east-1.amazonaws.com/prod`)

### Step A5: Verify Cognito Login Pages (already configured in Lab 1)

If you followed Lab 1, the login pages and domain are already set up. Verify:

1. Go to **Cognito Console** → Select your user pool
2. Go to **App clients** in the left sidebar → Click your app client
3. Click the **Login pages** tab and confirm:
   - **Allowed callback URLs:** `http://localhost:3000`
   - **Allowed sign-out URLs:** `http://localhost:3000`
   - **OAuth 2.0 grant types:** ✅ Implicit grant is checked
   - **OpenID Connect scopes:** ✅ openid, ✅ email, ✅ profile
4. If any of these are missing, click **Edit**, update them, and click **Save changes**
5. In the left sidebar, click **Domain** — note the full domain URL
   - It was auto-created and looks like: `https://abc123def.auth.YOUR_REGION.amazoncognito.com`

---

## Part B — Build the React App

### Step B1: Create the React Project

Open a terminal on your laptop:

```bash
npx create-react-app dost-ptri-day4-app
cd dost-ptri-day4-app
```

### Step B2: Create the Config File

This file does NOT exist yet — you need to create it.

1. Inside the `dost-ptri-day4-app/src/` folder, create a new file called `config.js`
2. Paste the following and replace all values with your actual AWS resource values:

```javascript
// REPLACE all values with your actual AWS resource values
const config = {
  // From API Gateway (Step A4)
  apiUrl: "https://YOUR_API_ID.execute-api.YOUR_REGION.amazonaws.com/prod",

  // From Cognito User Pool — Domain (auto-created, find it under Domain in left sidebar)
  cognitoDomain: "https://YOUR_COGNITO_DOMAIN.auth.YOUR_REGION.amazoncognito.com",
  clientId: "YOUR_COGNITO_APP_CLIENT_ID",
  redirectUri: "http://localhost:3000",
};

export default config;
```

### Step B3: Replace `src/App.js`

Replace the entire contents of `src/App.js` with:

```javascript
import { useState, useEffect, useCallback } from "react";
import config from "./config";

// Parse the token from the URL hash after Cognito redirect
function getTokenFromUrl() {
  const hash = window.location.hash.substring(1);
  const params = new URLSearchParams(hash);
  return params.get("id_token");
}

// Decode JWT payload (for display only, not for verification)
function decodeToken(token) {
  try {
    const payload = token.split(".")[1];
    return JSON.parse(atob(payload));
  } catch {
    return null;
  }
}

export default function App() {
  const [token, setToken] = useState(null);
  const [user, setUser] = useState(null);
  const [products, setProducts] = useState([]);
  const [cart, setCart] = useState([]);
  const [message, setMessage] = useState("");
  const [orders, setOrders] = useState([]);

  // Check for token on load
  useEffect(() => {
    const t = getTokenFromUrl();
    if (t) {
      setToken(t);
      setUser(decodeToken(t));
      window.history.replaceState({}, "", "/");
    }
  }, []);

  // Fetch products when logged in
  const fetchProducts = useCallback(async () => {
    if (!token) return;
    try {
      const res = await fetch(`${config.apiUrl}/items`, {
        headers: { Authorization: token },
      });
      const data = await res.json();
      setProducts(data.products || []);
    } catch (err) {
      setMessage("Error fetching products: " + err.message);
    }
  }, [token]);

  useEffect(() => {
    fetchProducts();
  }, [fetchProducts]);

  // Login redirect
  const login = () => {
    const url =
      `${config.cognitoDomain}/login?` +
      `client_id=${config.clientId}` +
      `&response_type=token` +
      `&scope=openid+email+profile` +
      `&redirect_uri=${encodeURIComponent(config.redirectUri)}`;
    window.location.href = url;
  };

  // Logout
  const logout = () => {
    setToken(null);
    setUser(null);
    setProducts([]);
    setCart([]);
    const url =
      `${config.cognitoDomain}/logout?` +
      `client_id=${config.clientId}` +
      `&logout_uri=${encodeURIComponent(config.redirectUri)}`;
    window.location.href = url;
  };

  // Add to cart
  const addToCart = (product) => {
    setCart([...cart, product]);
    setMessage(`Added ${product.name} to cart`);
  };

  // Place order (calls API → SQS + SNS)
  const placeOrder = async () => {
    if (cart.length === 0) {
      setMessage("Cart is empty!");
      return;
    }

    const total = cart.reduce((sum, item) => sum + item.price, 0);

    try {
      const res = await fetch(`${config.apiUrl}/orders`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          Authorization: token,
        },
        body: JSON.stringify({
          items: cart.map((item) => item.name),
          total: total,
        }),
      });
      const data = await res.json();
      setOrders([...orders, data.order]);
      setCart([]);
      setMessage(
        `✅ Order ${data.order.order_id} placed! Sent to SQS queue and SNS topic.`
      );
    } catch (err) {
      setMessage("Error placing order: " + err.message);
    }
  };

  // ── Styles ──
  const styles = {
    app: {
      fontFamily: "Arial, sans-serif",
      maxWidth: 800,
      margin: "0 auto",
      padding: 20,
    },
    header: {
      background: "#232f3e",
      color: "white",
      padding: "15px 20px",
      borderRadius: 8,
      display: "flex",
      justifyContent: "space-between",
      alignItems: "center",
    },
    btn: {
      padding: "8px 16px",
      border: "none",
      borderRadius: 4,
      cursor: "pointer",
      fontWeight: "bold",
    },
    btnPrimary: { background: "#ff9900", color: "#232f3e" },
    btnDanger: { background: "#d9534f", color: "white" },
    btnSuccess: { background: "#5cb85c", color: "white" },
    card: {
      border: "1px solid #ddd",
      borderRadius: 8,
      padding: 15,
      margin: "10px 0",
      display: "flex",
      justifyContent: "space-between",
      alignItems: "center",
    },
    alert: {
      background: "#fff3cd",
      border: "1px solid #ffc107",
      padding: 10,
      borderRadius: 4,
      margin: "10px 0",
    },
    section: { margin: "20px 0" },
  };

  // ── Not logged in ──
  if (!token) {
    return (
      <div style={styles.app}>
        <div style={{ textAlign: "center", marginTop: 100 }}>
          <h1>🛒 DOST PTRI Order System</h1>
          <p>Day 4 Lab — API Gateway + Cognito + SQS + SNS</p>
          <button
            style={{ ...styles.btn, ...styles.btnPrimary, fontSize: 18 }}
            onClick={login}
          >
            Login with Cognito
          </button>
        </div>
      </div>
    );
  }

  // ── Logged in ──
  return (
    <div style={styles.app}>
      {/* Header */}
      <div style={styles.header}>
        <div>
          <strong>🛒 DOST PTRI Order System</strong>
          <span style={{ marginLeft: 15, fontSize: 14 }}>
            Welcome, {user?.email || user?.["cognito:username"]}
          </span>
        </div>
        <button
          style={{ ...styles.btn, ...styles.btnDanger }}
          onClick={logout}
        >
          Logout
        </button>
      </div>

      {/* Status message */}
      {message && <div style={styles.alert}>{message}</div>}

      {/* Products */}
      <div style={styles.section}>
        <h2>📦 Products (from API Gateway → Lambda)</h2>
        {products.map((p) => (
          <div key={p.id} style={styles.card}>
            <div>
              <strong>{p.name}</strong>
              <span style={{ marginLeft: 10, color: "#666" }}>
                ₱{p.price.toLocaleString()}
              </span>
            </div>
            <button
              style={{ ...styles.btn, ...styles.btnPrimary }}
              onClick={() => addToCart(p)}
            >
              Add to Cart
            </button>
          </div>
        ))}
      </div>

      {/* Cart */}
      <div style={styles.section}>
        <h2>
          🛒 Cart ({cart.length} items — ₱
          {cart.reduce((s, i) => s + i.price, 0).toLocaleString()})
        </h2>
        {cart.length === 0 ? (
          <p style={{ color: "#999" }}>Cart is empty</p>
        ) : (
          <>
            {cart.map((item, i) => (
              <div key={i} style={{ padding: "4px 0" }}>
                • {item.name} — ₱{item.price.toLocaleString()}
              </div>
            ))}
            <button
              style={{
                ...styles.btn,
                ...styles.btnSuccess,
                marginTop: 10,
                fontSize: 16,
              }}
              onClick={placeOrder}
            >
              Place Order (→ SQS + SNS)
            </button>
          </>
        )}
      </div>

      {/* Order History */}
      {orders.length > 0 && (
        <div style={styles.section}>
          <h2>📋 Orders Placed This Session</h2>
          {orders.map((o, i) => (
            <div key={i} style={styles.card}>
              <div>
                <strong>{o.order_id}</strong> — {o.items.join(", ")} — ₱
                {o.total.toLocaleString()}
                <div style={{ fontSize: 12, color: "#999" }}>
                  {o.timestamp}
                </div>
              </div>
              <span style={{ color: "green" }}>✅ Sent</span>
            </div>
          ))}
        </div>
      )}

      {/* Architecture info */}
      <div
        style={{
          ...styles.section,
          background: "#f0f0f0",
          padding: 15,
          borderRadius: 8,
          fontSize: 14,
        }}
      >
        <h3>🏗️ What's happening behind the scenes:</h3>
        <ol>
          <li>
            <strong>Login</strong> → Cognito Hosted UI → returns JWT token
          </li>
          <li>
            <strong>Load Products</strong> → GET /items → API Gateway
            (validates token) → Lambda
          </li>
          <li>
            <strong>Place Order</strong> → POST /orders → API Gateway →
            Lambda → SQS Queue + SNS Topic
          </li>
          <li>
            <strong>SNS Fan-Out</strong> → Email Queue → Email Consumer
            Lambda
          </li>
          <li>
            <strong>SNS Fan-Out</strong> → Inventory Queue → Inventory
            Consumer Lambda
          </li>
        </ol>
      </div>
    </div>
  );
}
```

### Step B4: Replace `src/index.js`

Replace with a minimal version:

```javascript
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";

ReactDOM.createRoot(document.getElementById("root")).render(<App />);
```

### Step B5: Run the App

```bash
npm start
```

The app opens at **http://localhost:3000**.

---

## Part C — Test the Full Flow

### Test 1: Login
1. Click **Login with Cognito**
2. You'll be redirected to the Cognito managed login page
3. Log in with your test user from Lab 1 (your email / `MyNewPass123!`)
4. You'll be redirected back to the app, now logged in

### Test 2: Browse Products
- The product list loads automatically via **GET /items** through API Gateway
- Your JWT token is sent in the `Authorization` header
- The Lambda returns the product catalog

### Test 3: Place an Order
1. Click **Add to Cart** on a few products
2. Click **Place Order (→ SQS + SNS)**
3. You should see: `✅ Order ORD-XXXXXX placed! Sent to SQS queue and SNS topic.`

### Test 4: Verify in AWS Console
1. **SQS Console** → Check `dost-ptri-day4-[yourname]-order-queue` → Messages should have been received
2. **CloudWatch Logs** → Check the email consumer and inventory consumer Lambdas — they should show the order details
3. **CloudWatch Logs** → Check `dost-ptri-day4-[yourname]-app-backend` to see the API requests

---

## Troubleshooting

### "Unauthorized" error when loading products
- Your token may have expired (Cognito tokens last 1 hour). Click Logout and Login again.
- Verify the Cognito authorizer is attached to both GET `/items` and POST `/orders`.

### CORS errors in browser console
- Make sure you enabled CORS on both `/items` and `/orders` resources in API Gateway (Step A3).
- Make sure you **re-deployed** the API after enabling CORS (Step A4).
- The Lambda also returns CORS headers — check that `Access-Control-Allow-Origin: *` is in the response.

### "Login with Cognito" goes to an error page
- Check the **Allowed callback URLs** in the app client's **Login pages** tab includes exactly `http://localhost:3000` (no trailing slash).
- Make sure you created a Cognito domain (check **Domain** in the left sidebar).

### Products don't load after login
- Open browser DevTools (F12) → Network tab → check the `/items` request
- Verify `config.js` has the correct API URL and Client ID.

---

## What You Learned

- **Cognito Managed Login** handles user signup/login and returns JWT tokens — no need to build login forms
- **API Gateway + Cognito Authorizer** validates tokens automatically before your Lambda runs
- A single **React app** can integrate multiple AWS services through API Gateway
- **Place Order** triggers a chain: API Gateway → Lambda → SQS (async processing) + SNS (fan-out notifications)
- The entire backend is **serverless** — no servers to manage

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│  React App (localhost:3000)                                     │
│                                                                 │
│  [Login] ──→ Cognito Hosted UI ──→ JWT Token                   │
│                                                                 │
│  [Products] ──→ GET /items ──→ API Gateway ──→ Lambda           │
│                                    │                            │
│  [Place Order] ──→ POST /orders ──→ API Gateway ──→ Lambda      │
│                                                     │    │      │
│                                                     ▼    ▼      │
│                                                    SQS  SNS     │
│                                                          │      │
│                                                    ┌─────┴────┐ │
│                                                    ▼          ▼ │
│                                              Email Queue  Inv Q │
│                                                    ▼          ▼ │
│                                              Email λ    Inv λ   │
└─────────────────────────────────────────────────────────────────┘
```

## Cleanup

1. Stop the React app (`Ctrl+C` in terminal)
2. **Lambda:** Delete `dost-ptri-day4-[yourname]-app-backend`
3. All other resources — clean up using the instructions in Labs 1, 2, and 3

---

**← Previous:** [03 - SNS Fan-Out Notifications](./03-sns-fanout-notifications.md)
