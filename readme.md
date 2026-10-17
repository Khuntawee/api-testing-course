# 🍜 API Testing Lab: Auntie Som’s Noodle Stall

> **Objective:** Prove that Nephew Lek’s "it works on my machine" attitude is a recipe for disaster.

---

## 📖 The Backstory

Auntie Som runs a **legendary noodle stall**. Her Tom Yum is world-class, her customers are loyal, and her business is booming.  

Recently, her nephew **Lek** (who just finished a 2-week coding bootcamp) decided to "digitize" the business. He built a backend API to handle online orders. Auntie Som is thrilled, but the code… well, Lek says:  

> *"It’s fine lah Auntie! I tested it once with my own phone. No need for professional testing!"*

**You are the Last Line of Defense.**  
Before Auntie Som launches this app to 1,000+ hungry customers, you must audit the system, find the logic flaws, and automate the validation using **Bruno**.

---

## 🎯 Learning Objectives

By the end of this lab, you should be able to:
- 🖋️ Write **Bruno test scripts** for automated validation.
- 📦 Extract values from API responses using **Post-response scripts**.
- 🔐 Manage and use **Environment Variables**.
- 🧠 Verify **Business Rules**, not just HTTP status codes.
- 🚀 Catch regressions automatically before they hit production.

---

## 🛠️ Setup Instructions

### 1️⃣ Prepare the Project
- **Fork this repository** to your own GitHub account.
- **Clone it** locally to your machine.

### 2️⃣ Start the API Server
The backend is a lightweight Flask application.
- **Requirements:** Python 3.10+ installed.
- **Navigate** to the `auntie-som-noodle-api` folder.
- **Install dependencies:**  
  ```bash
  pip install flask
  ```
- **Run the server:**  
  ```bash
  python app.py
  ```
- The server will run at: `http://localhost:5000`

### 3️⃣ Setup Bruno
- Download and install [Bruno](https://www.usebruno.com).
- Create a **New Collection** named `Auntie Som Lab`.
- Create an **Environment** (e.g., `local`) and set a variable `baseUrl` to `http://localhost:5000`.

---

## 📑 API Reference

### 🔐 Authentication
`POST /auth/login` - Get an access token.
- **Body (JSON):**
  ```json
  {
    "username": "student",
    "password": "hungry"
  }
  ```
- **Response (200 OK):**
  ```json
  {
    "accessToken": "uuid-token-string",
    "expiresIn": 3600
  }
  ```

---

### 🍜 Menu
`GET /menu` - View available items and stock levels.
- **Response (200 OK):**
  ```json
  [
    { "id": 1, "name": "Tom Yum Noodles", "price": 50, "stock": 2 },
    { "id": 2, "name": "Fried Rice", "price": 45, "stock": 1 }
  ]
  ```

---

### 🛒 Orders
`POST /orders` - Place a new order.
- **Headers:** `Authorization: Bearer <token>`
- **Body (JSON):**
  ```json
  {
    "itemId": 1,
    "quantity": 1
  }
  ```
- **Response (200 OK):**
  ```json
  {
    "orderId": "ORD-abc123",
    "status": "created",
    "totalPrice": 45
  }
  ```

`GET /orders/<orderId>` - Retrieve order details.
- **Response (200 OK):**
  ```json
  {
    "orderId": "ORD-abc123",
    "status": "created",
    "totalPrice": 45
  }
  ```

---

## 🕵️ Your Mission: The Silent Auditor

Your task is to create a comprehensive Bruno collection that validates the entire workflow.  

⚠️ **The Catch:** Nephew Lek left several **logic bugs** in the system. Some endpoints might return `200 OK` even when the business logic is completely broken.

**Your collection must include:**
1.  **Auth Flow**: Automatically store the `accessToken` from login and use it for subsequent requests.
2.  **Order Validation**: Ensure orders can only be placed with valid auth, valid items, and available stock.
3.  **Data Integrity**: Check that the `totalPrice` calculated by the API matches your expectations.
4.  **Edge Cases**: What happens if you order more than what's in stock? What happens if you request a non-existent order?

> [!TIP]
> **Clicking "Send" is not testing.** Your tests must contain scripts (Assertions or Javascript) that **FAIL** when the system behaves incorrectly.

---

## 🧠 Rules of the Lab
- ❌ **Do NOT** modify the `app.py` backend code.
- ❌ **Do NOT** rely on manual checking.
- ✅ **Use environment variables** for the Base URL and Tokens.
- ✅ **Use Scripts/Assertions** for all validations.

---

## 📦 What to Submit

1.  **Push your Bruno collection folder** to your GitHub repository.
2.  **Update your README.md** (the one in your fork) with:
    - A summary of the bugs you discovered.
    - How your tests detect these bugs.
4.  **Send the GitHub link** to your instructor via Discord.

---
*Good luck. Auntie Som is counting on you!* 🍜🔥

---

## Local Bruno collection and audit results

The `auntie-som-noodle-api-test` collection contains ordered, automated tests for the login and order workflows. The original backend is unchanged. The checked-in example environment has no token; the login script captures a fresh token into the local environment.

### Set up and run

From the repository root, with Python 3.10+ and Bruno Desktop or the Bruno CLI installed:

```bash
python3 -m venv .venv
.venv/bin/python -m pip install 'Flask>=3.0,<4'
mkdir -p auntie-som-noodle-api-test/environments
cp auntie-som-noodle-api-test/local.bru.example auntie-som-noodle-api-test/environments/local.bru
.venv/bin/flask --app auntie-som-noodle-api/app.py run --host 127.0.0.1 --port 5000
```

Leave the server running. In another terminal, from the repository root:

```bash
cd auntie-som-noodle-api-test
bru run --env local
```

For the desktop app, open the `auntie-som-noodle-api-test` folder as a collection, select the `local` environment, and run the entire collection in sequence. The login and menu requests establish values used by later requests. The create-order and overstock scenarios modify in-memory stock: **stop and restart the API before every complete run**, including repeats in the desktop app. The initial stock checks detect a stale server. Tests expect a fresh server with the original menu. The expected total is calculated from the menu response and ordered quantity; the example total in the original API reference is inconsistent with its menu price.

Bruno may save the captured token to `environments/local.bru`. That machine-local file and `.venv` are ignored by Git; use the supplied example to recreate the environment. The CLI exits with a nonzero status while the backend defects below remain, which is expected for this audit.

### Results and reproduction

On 2026-10-17, Bruno CLI 4.2.1 and Flask 3.1.3 were run twice, each time against a freshly restarted server. Both runs completed all 13 requests: **8 tests passed and 5 failed**. The failures identify three backend defects:

| Defect | Reproduction and expected behavior | Observed behavior and detecting tests |
| --- | --- | --- |
| Incorrect order total | On a fresh server, log in, read the menu, and order two of item 1. Its menu price is 50, so the total should be 100. | Create and retrieve both return 95. `07 Create valid order` and `08 Retrieve created order` compare against the menu price times quantity. |
| Overstock accepted and inventory corrupted | On a fresh server, log in and order two of item 2, whose stock is 1. The order should be rejected with 400 and stock should remain 1. | The API returns 200 and the next menu shows stock -1. `11 Reject order above available stock` and `12 Verify stock after rejected overstock` check both effects. |
| Missing order reported as successful | Request `GET /orders/ORD-does-not-exist`. A missing order should return 404. | The API returns 200 with an error body. `13 Reject nonexistent order` checks the status. |

The other checks passed: login/token capture, invalid login, initial menu data, missing and invalid authentication, unknown item rejection, stock reduction after a valid order, and rejection of an already out-of-stock item. These results describe the original API; no fixes have been applied to it.
