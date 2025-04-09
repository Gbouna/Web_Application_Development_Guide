# Database and Storage Setup for Photo/Video Sharing App

Below is a **practical, step-by-step guide** for setting up your **database** and **storage** on Azure for the photo/video sharing application. This guide assumes you have already completed or are familiar with **Step 1 (Setting Up Azure Resources)**—meaning you have a resource group and Azure services ready.

---

## Overview

1. **Create / Configure a Database**  
   - Decide whether you want a **relational database (Azure SQL)** or **NoSQL database (Azure Cosmos DB)**.  
   - Create tables or containers to store user data, media metadata, comments, etc.

2. **Configure Azure Storage for Media**  
   - Use **Azure Blob Storage** for uploading and retrieving images/videos.  
   - Create containers and retrieve connection strings that your API will use.

Both of these pieces will ensure your application can handle data (e.g., user info, metadata) and large files (photos, videos).

---

## Part A: Database Setup

### Option 1: **Azure SQL Database (Relational)**

#### 1. Create (or Identify) an Azure SQL Database  
   - If you have not yet created one, follow the portal steps:  
     - In the [Azure Portal](https://portal.azure.com) → search for **"SQL databases"** → **+ Create**.  
     - Choose your resource group (e.g., `MyMediaAppRG`), name your database (e.g., `MyMediaDatabase`), and set up or reuse a SQL Server (e.g., `mymedia-sqlserver`).  

#### 2. Get the Connection String  
   - In the **SQL databases** list, click on your new database (`MyMediaDatabase`).  
   - In the left menu, select **Connection strings** (or "Show database connection strings").  
   - Copy the **ADO.NET connection string**. It will look like:  
     ```text
     Server=tcp:mymedia-sqlserver.database.windows.net,1433;
     Initial Catalog=MyMediaDatabase;
     Persist Security Info=False;
     User ID=YourSQLAdmin;
     Password=YourPassword;
     MultipleActiveResultSets=False;
     Encrypt=True;
     TrustServerCertificate=False;
     Connection Timeout=30;
     ```
   - You'll eventually place this string in your API code or in Azure App Settings so your code can connect to the database.

#### 3. Allow Your App to Access the Database  
   - Under **"Firewall and virtual networks"** (for your SQL Server), add your current client IP address if you want to manage the database from your local machine.  
   - For your App Service to connect, you usually won't need to do anything extra unless you've restricted the server's access. (By default, Azure App Services can connect to Azure SQL in the same subscription.)

#### 4. Create Database Tables (Schema)  
   - To create tables for your app (e.g., Users, Media, Comments), you have two main approaches:  

     **1. Azure Portal Query Editor:**  
        - In the Azure Portal, open your SQL database → "Query editor (preview)" (left menu).  
        - Sign in with your SQL credentials.  
        - Run SQL statements like:
          ```sql
          CREATE TABLE Users (
              UserId INT PRIMARY KEY IDENTITY,
              Username VARCHAR(100) NOT NULL,
              Email VARCHAR(100) NOT NULL,
              Role VARCHAR(50) NOT NULL  -- e.g., 'creator' or 'consumer'
          );

          CREATE TABLE Media (
              MediaId INT PRIMARY KEY IDENTITY,
              Title VARCHAR(200) NOT NULL,
              Caption VARCHAR(MAX),
              Location VARCHAR(200),
              UploadedBy INT NOT NULL,  -- references Users.UserId
              CONSTRAINT FK_Media_Users FOREIGN KEY (UploadedBy) REFERENCES Users(UserId)
          );

          CREATE TABLE Comments (
              CommentId INT PRIMARY KEY IDENTITY,
              MediaId INT NOT NULL,
              UserId INT NOT NULL,
              CommentText VARCHAR(MAX),
              CONSTRAINT FK_Comments_Media FOREIGN KEY (MediaId) REFERENCES Media(MediaId),
              CONSTRAINT FK_Comments_Users FOREIGN KEY (UserId) REFERENCES Users(UserId)
          );
          ```

     **2. Local SQL Tool:**  
        - Install a desktop client like **Azure Data Studio** or **SQL Server Management Studio (SSMS)**.  
        - Connect to your SQL Server with the same credentials.  
        - Run the **CREATE TABLE** statements locally to build the schema.

**Result**: You have a relational database with tables for users, media items, and comments. You can add more tables (like Ratings) or columns (like timestamps) to suit your needs.

---

### Option 2: **Azure Cosmos DB (NoSQL)**

If you prefer a NoSQL approach:

#### 1. Create (or Identify) a Cosmos DB Account  
   - In the Azure Portal, search for **"Azure Cosmos DB"** → **+ Create**.  
   - Choose the API type (e.g., **Core (SQL) API**), resource group, name, region, and free-tier/serverless options if available.

#### 2. Create a Database and Container  
   - Go to your Cosmos DB account in the portal, and look for **"Data Explorer"** on the left.  
   - Click **"New Database"**. For example:  
     - Database Id: `MyMediaDB`  
     - Throughput: If asked, keep it minimal or "Pay as you go" (serverless).  
   - Inside that database, create a **New Container** for your data. E.g.,  
     - Container Id: `MediaItems`  
     - Partition Key: `/MediaId` or a logical partition like `/UploadedBy` (depending on your data usage).  
   - You can create multiple containers, for instance:  
     - `Users`, `MediaItems`, `Comments` (each container holding a different type of data).

#### 3. Get the Connection String  
   - In the left menu, go to **"Keys"**. You'll see a **Primary Connection String**.  
   - You'll use this in your API code (e.g., environment variable or config file) so your code can talk to Cosmos DB.

#### 4. Insert Sample Data (Optional)  
   - In "Data Explorer," you can manually add JSON documents to test your setup. For example:
     ```json
     {
       "id": "mediaItem1",
       "title": "My First Video",
       "caption": "Check out my new video!",
       "location": "Paris",
       "uploadedBy": "user123",
       "type": "video"
     }
     ```
   - This helps confirm you can read/write data from the container.

**Result**: You have a NoSQL database set up with containers that store JSON documents for users, media, comments, etc.

---

## Part B: Azure Storage Setup for Media Files

#### 1. Locate Your Storage Account  
   - In the Azure Portal, search for **"Storage accounts"** and select the one you created (e.g., `mymediaappstorage123`).

#### 2. Create a Blob Container  
   - In the left menu, find **"Containers"**.  
   - Click **+ Container** to create a new container, e.g., `mediauploads`.  
   - **Public Access Level**: Usually "Private" to ensure files aren't directly publicly accessible. (You can generate secure download links later.)

#### 3. Get Your Storage Connection String  
   - Still in the Storage Account view, click on **"Access keys"** under **Security + networking** (or a similar section).  
   - You'll see "key1" and "key2" along with a **Connection string** for each.  
   - Copy the connection string for "key1." It looks like:
     ```text
     DefaultEndpointsProtocol=https;
     AccountName=mymediaappstorage123;
     AccountKey=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX;
     EndpointSuffix=core.windows.net
     ```
   - You'll use this in your API to upload and retrieve blobs programmatically.

#### 4. Uploading a Test File (Optional)  
   - Click on the container (`mediauploads`) → "Upload."  
   - Select a small image or test file from your computer.  
   - Verify it uploads successfully.

#### 5. Generating a SAS (Shared Access Signature) URL (Optional)  
   - To share or test direct access to a file, you can generate a "Shared Access Signature" link.  
   - In your container, right-click (or click the **...**) on the blob and select **"Generate SAS"**.  
   - Set the expiry date/time and permissions (e.g., read).  
   - Copy the generated URL and test it in your browser.

**Result**: Your Azure Blob Storage is ready to store actual media files. You can connect your API to it using the connection string to handle uploads and downloads.

---

## Common Post-Setup Tasks

1. **Configure Your App (API) with Connection Strings**  
   - In your **Web App** (App Service), go to **"Configuration"** → **"Application settings"**.  
   - Add new settings like `SQL_CONNECTION_STRING` or `COSMOS_CONNECTION_STRING`, `STORAGE_CONNECTION_STRING`, and paste the appropriate values.  
   - Your code can read these environment variables securely, instead of hardcoding secrets in code.

2. **Security Best Practices**  
   - **Never** commit connection strings or keys to public GitHub repositories.  
   - Use **Azure Key Vault** for extra security (optional, more advanced).

3. **Create Additional Tables/Containers**  
   - If you decide to add more data features (e.g., a separate Ratings table/container), follow the same steps to define your structure.

4. **Plan for Growth**  
   - If your usage grows, you might need to adjust the **tier** or **throughput** of your database or storage.

---

## Wrapping Up

By completing these steps, you should have:

1. **A Database** (SQL or NoSQL) to store user profiles, media metadata, comments, etc.  
2. **Blob Storage** ready to handle large files (photos or videos).  
3. **Connection Strings** that your API can use to read/write data securely.  

**Next Steps**: Move on to building your RESTful API (Step 3). You'll use the connection strings from your database and storage account to enable create/read/update/delete (CRUD) operations for your media data and handle file uploads/downloads.

---
