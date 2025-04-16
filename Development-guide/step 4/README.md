# Azure AD B2C Authentication & Authorization Guide

A **step-by-step, beginner-friendly guide** to implement **Authentication (Auth)** and **Authorization** in your application using **Azure AD B2C** (Business-to-Consumer). This ensures only authorized users (e.g., "creator" vs. "consumer") can access or modify specific features.

---

## Overview

1. **Create an Azure AD B2C Tenant**: Set up a directory for user identities.  
2. **Register Applications in B2C**: Create entries for your API and front-end.  
3. **Set Up User Flows**: Configure sign-up/sign-in policies.  
4. **Integrate B2C with Your Back-End**: Protect API endpoints with token validation.  
5. **Role-Based Authorization**: Assign and enforce roles (e.g., "creator" or "consumer").  

> **Note**: Code examples use Node.js/Express with `passport-azure-ad`, but concepts apply to other frameworks.

---

## Part A: Create an Azure AD B2C Tenant

### 1. Check for Existing B2C Tenant
- Log in to the [Azure Portal](https://portal.azure.com).  
- Search for **Azure AD B2C**.  
- Reuse an existing tenant or create a new one.

### 2. Create a New Tenant
- Click **Create** and fill in:  
  - **Organization name**: `MyMediaAppB2C`.  
  - **Initial domain name**: `my-mediaapp-b2c.onmicrosoft.com` (must be unique).  
  - **Country/region**: Your location.  

### 3. Link to Azure Subscription
- In the B2C directory, navigate to **Azure AD B2C** → **Manage tenant** → **Linked subscriptions**.  
- Associate with your Azure subscription (e.g., Azure for Students).  

---

## Part B: Register Applications

### 1. Register the API Application
1. Go to **App Registrations** → **New Registration**.  
2. **Name**: `MyMediaAPI`.  
3. **Supported account types**: Default ("Accounts in this organizational directory only").  
4. **Redirect URI**: Optional for APIs (skip if front-end handles login).  
5. Click **Register**.  

**After registration**:  
- Copy the **Application (client) ID** and **Directory (tenant) ID**.  
- Under **Certificates & secrets**, generate a **client secret** if needed.  

### 2. (Optional) Register a Front-End App
1. **New Registration** → **Name**: `MyMediaFrontEnd`.  
2. **Redirect URI**: Add your front-end URL (e.g., `https://my-front-end.azurewebsites.net`).  
3. Save the **Client ID**.  

---

## Part C: Set Up User Flows

1. Navigate to **Azure AD B2C** → **User flows**.  
2. **New User Flow** → Select **"Sign up and sign in"**.  
3. **Name**: `B2C_1_SignUpSignIn`.  
4. **Identity providers**: Email/username (or add Google/Facebook).  
5. **User attributes**: Choose fields (e.g., display name, email).  
6. Click **Create**.  

> **Note**: The **Policy Name** (e.g., `B2C_1_SignUpSignIn`) is used later for integration.

---

## Part D: Integrate B2C with Your Back-End

### Node.js/Express Example

#### 1. Install Dependencies
```bash
npm install passport passport-azure-ad
