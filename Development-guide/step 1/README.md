# Azure Resource Setup Guide

Below is a **step-by-step manual** for setting up Azure resources in a way that is approachable for anyone—especially those new to cloud computing. The goal is to get you from "no Azure resources" to a well-organized environment where you can deploy your application.

---

## Step 0: **Prerequisites**

1. **Azure for Students Subscription**  
   - Sign up for an [Azure for Students](https://azure.microsoft.com/free/students/) account if available in your country/region.  
   - This typically gives you credits to use Azure services without needing a credit card, plus free-tier services.  
   - If Azure for Students is not available, create a regular free Azure account instead.

2. **Azure Portal Access**  
   - Ensure you can log into the [Azure Portal](https://portal.azure.com). This is where you'll manage your resources.

> **Note**: The steps below apply to the Azure Portal's web interface. The layout might occasionally change, but the process should be similar.

---

## Step 1: **Create a Resource Group**

A Resource Group is like a "folder" that holds all the components (services, databases, etc.) for your project in one place.

1. **Log In**  
   - Go to the Azure Portal ([portal.azure.com](https://portal.azure.com)) and sign in with your Azure account.

2. **Search for "Resource groups"**  
   - In the top search bar, type **"Resource groups"** and select the matching service from the dropdown.

3. **Add a Resource Group**  
   - Click the **"+ Create"** or **"+ Add"** button (the wording can vary, but you're looking for a "create" or "add" button).  
   - You will see a form to create a new resource group.

4. **Fill Out the Form**  
   - **Subscription**: Make sure it shows your "Azure for Students" subscription.  
   - **Resource Group Name**: Choose a descriptive name, e.g. `MyMediaAppRG`. (Use something short and clear.)  
   - **Region**: Pick a location close to you or your audience (e.g., `East US`, `West Europe`).  
   - Click **Review + Create**, then **Create** again to finalize.

> **Result**: You now have a resource group named `MyMediaAppRG` in your chosen region. All subsequent resources for this project should be created within this group.

---

## Step 2: **Create an App Service Plan**

An App Service Plan is like the "machine" or "virtual environment" that will run your web apps (e.g., your REST API, or even your front-end if you choose).

1. **Search "App Service plans"**  
   - In the Azure Portal search bar, type **"App Service plans"**.

2. **Create App Service Plan**  
   - Click **"+ Create"**.  
   - **Subscription**: Again, confirm it's under "Azure for Students."  
   - **Resource Group**: Select the resource group you created (`MyMediaAppRG`).  
   - **Name**: Something like `MyMediaAppPlan`.  
   - **Region**: The same region you chose for your resource group if possible.  
   - **Pricing Tier**:  
     - Click **Change size**.  
     - Look for a **Free** or **Shared** tier (often labeled "F1" or "D1"), depending on what's available under the student subscription.  

3. **Review + Create**  
   - After selecting the Free tier, click **Review + Create**.  
   - Then click **Create** to finalize.

> **Result**: You now have an **App Service Plan** that will host your web apps at no or minimal cost.

---

## Step 3: **Create an App Service (Web App)**

This is where your actual application (front-end or REST API) can live.

1. **Search "App Services"**  
   - In the Azure Portal search bar, type **"App Services"**.

2. **Create a Web App**  
   - Click **"+ Create"**.  
   - **Subscription**: "Azure for Students."  
   - **Resource Group**: `MyMediaAppRG`.  
   - **Name**: Must be globally unique (e.g. `my-media-app-unique`).  
   - **Publish**: Select "Code" if you plan to deploy a Node.js, Python, or other code-based API. If you have a container, choose "Docker Container" (probably not necessary for first-time users).  
   - **Runtime Stack**: For example, if you're using Node.js, choose the Node version. If you plan a .NET app, select .NET.  
   - **Region**: Match your existing region.  
   - **App Service Plan**: Select the one you created, `MyMediaAppPlan`.

3. **Review + Create**  
   - Click **Review + Create** and then **Create**.

> **Result**: You have a **Web App** under your **App Service Plan**, ready to host your code. You can test it by going to the URL Azure assigns, e.g., `https://my-media-app-unique.azurewebsites.net`.

---

## Step 4: **Create a Storage Account (for Media Files)**

You need a place to store uploaded images or videos. Azure Storage Accounts let you create **Blob Storage**, which is perfect for media files.

1. **Search "Storage accounts"**  
   - Click **"Storage accounts"** in the search results.

2. **Create a Storage Account**  
   - Click **"+ Create"**.  
   - **Subscription**: "Azure for Students."  
   - **Resource Group**: `MyMediaAppRG`.  
   - **Storage account name**: Must be globally unique, e.g., `mymediaappstorage123`.  
   - **Region**: Same region as before.  
   - **Performance**: Standard (cheaper and perfectly fine for most needs).  
   - **Redundancy**: Locally-redundant storage (LRS) is typically the cheapest.  
   - Keep other defaults.

3. **Review + Create**  
   - Click **Review + Create** then **Create**. 
   - Wait for the deployment to finish.

> **Result**: You now have a **Storage Account**. Inside it, you'll create a Blob Container to hold your media.

### **(Optional Sub-Step)**: Create a Blob Container
1. Go to your new Storage Account (e.g., `mymediaappstorage123`).  
2. On the left menu, find **"Containers"** and click **"+ Container"**.  
3. **Name**: Something like `videos` or `images`.  
4. **Public Access**: Typically set to "Private (no anonymous access)" so that only authorized requests can get the files. (You can generate secure URLs later from your API.)  
5. Click **Create**.

---

## Step 5: **Create a Database (Azure SQL or Cosmos DB)**

You need a database to store user data, metadata, comments, etc. The choice depends on whether you prefer **relational** (Azure SQL) or **NoSQL** (Cosmos DB).

### 5A: **Azure SQL Database (Relational)**
1. **Search "SQL databases"** in the portal.  
2. Click **"+ Create"** and fill out:
   - **Subscription**: "Azure for Students."  
   - **Resource Group**: `MyMediaAppRG`.  
   - **Database Name**: e.g., `MyMediaDatabase`.  
   - **Server**: 
     - Click **Create new** if you don't already have one.  
     - Provide a **server name**, e.g., `mymediadbserver`, region, and admin username/password (keep these noted!).  
   - **Compute + storage**: Pick the smallest tier (e.g., Basic or an equivalent free/cheap tier).  
3. **Review + Create** → **Create**.

> You'll have an admin login (e.g., `dbadmin`) and password for your SQL Database. You'll use these to connect your API to the database.

### 5B: **Azure Cosmos DB (NoSQL)**
1. **Search "Azure Cosmos DB"**.  
2. **+ Create** a new Cosmos DB account.  
3. **API**: You can choose Core (SQL) API, MongoDB API, etc. The Core (SQL) API is a popular choice.  
4. **Subscription**: "Azure for Students," **Resource Group**: `MyMediaAppRG`.  
5. **Account Name**: e.g. `mycosmosmedia`. Must be globally unique.  
6. **Location**: same region.  
7. **Capacity Mode**: "Serverless" if available or the free tier to minimize costs.  
8. **Review + Create** → **Create**.

> With Cosmos DB, you'll create a "Database" and "Container" inside your Cosmos account to hold your data.

---

## Step 6: **(Optional) Configure Identity Service (Azure AD B2C)**

If you plan to handle user sign-ups and logins, Azure AD B2C (Business to Consumer) is a good choice.

1. **Search "Azure AD B2C"** in the portal.  
2. Follow prompts to create an Azure AD B2C tenant (this is like a separate directory where consumer users will live).  
3. Link the new tenant to your subscription.  
4. Set up "User Flows" or "Custom Policies" depending on your needs.  
5. Obtain application/client IDs which you'll plug into your front-end or back-end to handle authentication flows.

> This step can be a bit more advanced; you can skip it if you plan to handle authentication differently or keep it simple (e.g., manual user data in your database for demonstration).

---

## Step 7: **Organize and Review All Resources**

To recap, you should now see the following items in your **MyMediaAppRG** resource group:

1. **App Service Plan** (`MyMediaAppPlan`)  
2. **Web App** (`my-media-app-unique.azurewebsites.net`)  
3. **Storage Account** (`mymediaappstorage123`) and potentially a **Blob Container**.  
4. **Azure SQL Database** or **Azure Cosmos DB** (for storing app data).  
5. **(Optional)** Azure AD B2C tenant for authentication.

---

## Step 8: **Next Steps**

Now that everything is created and organized, you can move forward with:

1. **Configuring Your Database** (step-by-step table creation, or container setup for Cosmos DB).  
2. **Writing Your REST API** and pointing it to the database and storage.  
3. **Deploying Your API** to the Web App you created.  
4. **Setting Up Authentication** with Azure AD B2C (if applicable).  
5. **Testing** each resource to make sure it's accessible.

---

## Common Tips & Troubleshooting

1. **Naming Conflicts**  
   - If Azure says the name is taken, pick a new globally unique name. 
2. **Cost Alerts**  
   - In your subscription settings, set up "cost alerts" or check the cost analysis regularly to ensure you don't exceed free limits.
3. **Resource Group Organization**  
   - Delete the entire resource group once you finish the project or no longer need it. This helps avoid leftover costs.
4. **Security**  
   - Don't share admin credentials. Store them in a safe place or use Azure Key Vault for secrets if needed.

---

### Final Note

By following these instructions, you should have all the **Azure building blocks** ready. The next big steps involve setting up your databases, writing/deploying your API, and building your front-end. This resource setup is the foundation on which the rest of your application will run.