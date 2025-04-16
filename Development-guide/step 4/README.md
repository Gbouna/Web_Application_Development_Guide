# Implementing Authentication and Authorisation with Azure AD B2C

Below is a **step-by-step, beginner-friendly guide** to implement **Authentication (Auth)** and **Authorization** in your application using **Azure AD B2C** (Business-to-Consumer). This will ensure that only the right users (e.g., “creator” vs. “consumer”) can access or modify certain features, like uploading media or commenting.

---

## Overview of the Process

1. **Create an Azure AD B2C Tenant**: Set up a separate directory in Azure for managing user identities.
2. **Register Applications in B2C**: Create an “application” entry that represents your back-end (API) and possibly your front-end.
3. **Set Up User Flows (Sign-up / Sign-in)**: Configure how users register and log in to your service.
4. **Integrate B2C with Your Back-End**: Protect your REST endpoints with token validation.
5. **Role-Based Authorization**: Decide how you assign roles (e.g., “creator” or “consumer”) and enforce them in your API.

This guide focuses on the **concepts** of how to plug in Azure AD B2C. The actual code can vary depending on your back-end framework (Node.js, .NET, Python, etc.). We’ll give examples in Node.js/Express with the **passport-azure-ad** library for JWT validation.

---

## Part A: **Create an Azure AD B2C Tenant**

1. **Check if B2C is Already Enabled**
   - Log in to the [Azure Portal](https://portal.azure.com).
   - Search for “Azure AD B2C.”
   - If you see an existing tenant, you can reuse it; otherwise, proceed to create one.

2. **Create a New B2C Tenant**
   - Click **Create** and fill in the required information:
     - **Organization name**: e.g. `MyMediaAppB2C`.
     - **Initial domain name**: e.g. `my-mediaapp-b2c.onmicrosoft.com` (must be unique).
     - **Country/region**: your location.
   - After creation, you’ll have a separate **B2C directory**. You can switch directories by clicking on your profile in the top-right corner of the portal.

3. **Link B2C Tenant to Your Azure Subscription**
   - In the B2C directory, navigate to **Azure AD B2C** → **“Manage tenant”** → **“Linked subscriptions”** if needed.
   - Make sure the B2C tenant is associated with your Azure for Students subscription so it can use the resources.

---

## Part B: **Register Your Applications**

### 1. **Register the Server/API Application**

1. **In the B2C Directory**, go to **App Registrations** → **New Registration**.
2. **Name**: e.g., `MyMediaAPI`.
3. **Supported account types**: Typically the default is fine for B2C (“Accounts in this organizational directory only”).
4. **Redirect URI** (optional for APIs): You only need this if you want direct user login flows from the API, which is less common. Usually your front-end handles the login and your API just validates tokens.
5. Click **Register**.

After registration:
- **Application (client) ID** will be displayed (copy it).
- **Directory (tenant) ID** (copy it as well).

Under **Certificates & secrets**, generate a **client secret** if you need the API to call B2C on behalf of users. For simply validating tokens, you typically only need the **tenant ID** and the **client ID**.

### 2. **(Optional) Register a Client/Web App**

If you have a separate front-end that handles user sign-in (e.g., a React app, or even a statically hosted site):
1. **New Registration** → **Name**: `MyMediaFrontEnd`.
2. For “Redirect URIs,” include the URL of your front-end app, e.g., `https://my-front-end.azurewebsites.net`.
3. You’ll get a **Client ID** for the front-end.

---

## Part C: **Create User Flows (Sign-up / Sign-in)**

1. **Go to** Azure AD B2C → **User flows** (under “Policies”).
2. **New User Flow** → Choose **“Sign up and sign in”** (or “Sign in” only if you plan to create accounts manually).
3. **Name**: e.g., `B2C_1_SignUpSignIn`.
4. **Identity providers**: Typically email/username. (You can also configure Google, Facebook, etc. if you want.)
5. **User attributes**: Choose which info you want from users (e.g., display name, email).
6. **Create**.

When done, you’ll see a **Policy Name** (like `B2C_1_SignUpSignIn`). This is important for your front-end or back-end to initiate user flows.

---
