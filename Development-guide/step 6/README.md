# Hosting your front-end

Below is a **step-by-step, guide** for **hosting your front-end** on Azure. We’ll cover two main approaches:

1. **Static Web Hosting** (Azure Storage or Azure Static Web Apps)  
2. **Azure App Service** (alongside or separate from your API)

These options ensure your front-end is publicly accessible at a stable URL so users can interact with your application.

---

## Option A: **Static Web Hosting**

### 1. **Using Azure Storage (Static Website Hosting)**

Azure Storage Accounts can serve static content (HTML, CSS, JS) directly to users:

1. **Create or Identify a Storage Account**  
   - From previous steps, you might already have a Storage Account. If not, create one.  
   - Make sure it’s in the same Resource Group as your project (e.g., `MyMediaAppRG`).

2. **Enable Static Website Hosting**  
   - In the Azure Portal, go to your Storage Account.  
   - In the left menu, find **“Static website”** (under “Data management” or similar).  
   - Click **“Enabled”** for “Static website.”  
   - It will ask for an **index document name** (e.g., `index.html`) and an **error document path** (e.g., `404.html`).  
   - Click **Save**.

3. **Upload Your Front-End Files**  
   - After enabling static website hosting, Azure automatically creates a special **$web** container.  
   - You can upload files in two main ways:  
     - **Via the Azure Portal**  
       - Go to **Storage Explorer** (in the portal) or **Containers** and find the **$web** container.  
       - Click **Upload** → select your `index.html`, CSS, JS files, etc.  
     - **AzCopy or Azure CLI** (more advanced)  
       - Install [AzCopy](https://learn.microsoft.com/azure/storage/common/storage-use-azcopy-v10) or use the [Azure CLI](https://learn.microsoft.com/cli/azure).  
       - Run commands to sync or upload your front-end folder to `$web`.

4. **Get Your Static Website URL**  
   - Back on the **Static website** page, Azure shows you a **Primary endpoint**, e.g. `https://<yourstorageaccount>.z13.web.core.windows.net`.  
   - Open that URL in your browser. You should see your `index.html` page.

#### Pros & Cons of Azure Storage
- **Pros**: Very simple, often cheaper, straightforward for purely static content.  
- **Cons**: No built-in automated build steps or server-side logic. If you need advanced routing or serverless functions, see the next sub-option or use your back-end App Service.

---

### 2. **Using Azure Static Web Apps**

Azure Static Web Apps is designed to host front-end files (HTML, CSS, JS) plus optional **serverless APIs** in Azure Functions. However, you already have an API in App Service, so you’ll likely use Static Web Apps just for your front-end.

1. **Sign in to the Azure Portal** → search for **“Static Web Apps”**.  
2. **+ Create** a Static Web App:  
   - **Subscription**: Use your Student subscription.  
   - **Resource Group**: `MyMediaAppRG` or similar.  
   - **Name**: Must be unique globally, e.g., `mymedia-frontend`.  
   - **Region**: Match or near your existing region.  
   - **Deployment Details**:  
     - If your code is on GitHub, you can connect to your repo for automatic deployments.  
     - If you don’t use GitHub, choose “Other” and do a manual deployment.  
3. **If Using GitHub**:  
   - You’ll be prompted to authorize Azure to access your repository.  
   - Select your repo and branch that contains your front-end (e.g., `main`).  
   - Specify the **App location** (the folder with `index.html`). If it’s the root, just put `/.`  
   - Skip API location if you don’t have an Azure Functions API in the same project.  
   - The build and output location can be `/.` if you’re not using any build tools.  
4. **After Creation**:  
   - You’ll get a URL like `https://mymedia-frontend.azurestaticapps.net`.  
   - Visit that URL in your browser. Your front-end should load.  
5. **Manual Deployment** (if not using GitHub)  
   - Once the Static Web App is created, go to its **“Deployment”** or **“Build”** section.  
   - You’ll see instructions to upload files. Typically, you can use the Azure CLI or GitHub Actions approach.  

#### Pros & Cons of Static Web Apps
- **Pros**: Simple, free tier available, integrated with GitHub for auto-deploy, can handle custom domains.  
- **Cons**: Requires some initial setup if you’re not familiar with GitHub Actions or manual deployment.

---

## Option B: **Azure App Service**

You can also host your front-end **on the same App Service** as your back-end, or on a **separate** App Service specifically for static files.

### 1. **Single App Service for Both API & Front-End**

1. **Locate Your Existing App Service** (from Step 2).  
2. **Add a `public` or `wwwroot` folder** to your Node.js (or .NET) project containing your HTML, CSS, JS.  
3. **Serve Static Files** in your back-end code:
   ```js
   // Example (Node/Express):
   const express = require('express');
   const path = require('path');
   const app = express();

   // Serve static files from 'public' folder
   app.use(express.static(path.join(__dirname, 'public')));

   // If your 'index.html' is in 'public', you can open it at https://<your-app>.azurewebsites.net/
   app.get('/', (req, res) => {
     res.sendFile(path.join(__dirname, 'public', 'index.html'));
   });
   ```
4. **Deploy** to your App Service (e.g., via GitHub or Zip Deploy).  
5. **Browse** to `https://<your-app>.azurewebsites.net/` and you’ll see your front-end.

**Note**: This approach merges front-end and back-end in one web app. Make sure your routes (e.g., `/api/...`) don’t conflict with static files.

---

### 2. **Separate App Service for the Front-End**

1. **Create a New App Service**  
   - Like you did for your API, but name it something like `MyMediaFrontEndApp`.
2. **Deploy** your static files the same way you’d deploy an API.  
   - If you’re using Node, you might just have a minimal `server.js` that serves your static files:
     ```js
     const express = require('express');
     const app = express();
     app.use(express.static('public')); // folder with your HTML
     app.listen(process.env.PORT || 3000);
     ```
3. **Visit** `https://<mymediafrontendapp>.azurewebsites.net/` to see your front-end.

---

## Configuration and Best Practices

1. **CORS (Cross-Origin Resource Sharing)**  
   - If your front-end is served from `https://<frontend>.azurestaticapps.net` and your API is at `https://<api>.azurewebsites.net`, you may need to enable CORS in your back-end so the browser can request data.  
   - In your API, set allowed origins or use `'*'` (less secure but simpler for testing).

2. **Custom Domain** (Optional)  
   - In either Static Web Apps or App Service, you can map a custom domain (e.g., `www.mymediaapp.com`).  
   - This involves verifying domain ownership via DNS and configuring Azure to use that domain.

3. **HTTPS**  
   - By default, Azure provides **HTTPS** on all `.azurewebsites.net` and `.azurestaticapps.net` domains.  
   - If you bring a custom domain, you can set up an SSL certificate, often free via Azure-managed certificates.

4. **Deploy Automation**  
   - If your code is on GitHub, you can set up CI/CD so that any push to the main branch automatically updates your app. This is especially handy with **Static Web Apps**.

---

## Testing & Troubleshooting

1. **Check the URL**  
   - Make sure you’re loading the correct endpoint in your web browser.  
   - Confirm your `apiBaseUrl` (in your front-end JavaScript) matches the actual deployed API URL.
2. **Open F12 Developer Tools** in your browser.  
   - Check **Console** for JavaScript errors.  
   - Check **Network** tab to see if your API calls are succeeding or failing (e.g., 404, 401).
3. **Azure Logs**  
   - For Static Web Apps, logs are found in **GitHub Actions** if you used that approach.  
   - For App Service, look at **Log Stream** or **App Insights** if you enabled it.
4. **CORS** issues often appear as “Blocked by CORS Policy” in the Console.  
   - Fix by adding the correct domain to your API’s CORS settings.

---

## Final Recap

By the end of **Step 7**, you should have:

1. **Front-End Hosted Publicly**  
   - Users can access your `index.html` page, search or view media, etc.
2. **Working Integration** with your REST API  
   - The front-end calls your Azure-hosted API endpoints to get or post data.
3. **Deployment** either as:  
   - **Static Web Hosting** (via Azure Storage or Azure Static Web Apps).  
   - **App Service** (shared with your API or a separate front-end App Service).

**Congratulations!** You now have a live, publicly accessible front-end for your photo/video sharing application. 
