# implementation of Authentication and Authorization

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

## Part D: **Integrate B2C Authentication in Your Back-End**

The simplest approach is to **validate tokens** (JWTs) in your API’s request headers. When a user logs in via your B2C sign-in page (initiated by your front-end), they’ll receive a JWT. That token is then sent in the **Authorization** header (e.g., `Bearer <token>`) for each request to your API.

### Node.js / Express Example

1. **Install `passport-azure-ad`**:
   ```bash
   npm install passport passport-azure-ad
   ```
2. **Configure a Bearer Strategy** in your back-end:
   ```js
   // auth.js
   const passport = require('passport');
   const BearerStrategy = require('passport-azure-ad').BearerStrategy;

   const options = {
     identityMetadata: `https://<your-tenant-name>.b2clogin.com/<your-tenant-name>.onmicrosoft.com/<your-policy>/v2.0/.well-known/openid-configuration`,
     clientID: '<CLIENT_ID_OF_YOUR_API>',
     // 'B2C_1_SignUpSignIn' or your user flow/policy name
     policyName: '<YOUR_POLICY_NAME>',
     isB2C: true,
     validateIssuer: true,
     loggingLevel: 'info',
     passReqToCallback: false
   };

   passport.use(new BearerStrategy(options, (token, done) => {
     // This function is called after token is validated
     // token contains user info
     done(null, token);
   }));

   module.exports = passport;
   ```
   - Replace `<your-tenant-name>` with your domain, like `my-mediaapp-b2c`.  
   - `<CLIENT_ID_OF_YOUR_API>` is from your API registration in B2C.  
   - `<YOUR_POLICY_NAME>` is something like `B2C_1_SignUpSignIn`.

3. **Use the Strategy in Your Express App**:
   ```js
   // index.js or app.js
   const express = require('express');
   const passport = require('./auth'); // the file where we set up BearerStrategy
   const app = express();
   app.use(passport.initialize());

   // A protected route example
   app.get('/api/protected', 
     passport.authenticate('oauth-bearer', { session: false }),
     (req, res) => {
       // If we're here, the token is valid
       res.send('You are authenticated!');
     }
   );
   ```
4. **Test**:
   - You’ll need a valid **Bearer token** from logging in through your B2C user flow.  
   - If you’re testing manually, you can create your own front-end or a tool like **Postman** to get the token from B2C.  
   - When you include `Authorization: Bearer <token>` in the header and call `/api/protected`, you should see `"You are authenticated!"`.

---

## Part E: **Role-Based Authorization**

Now you want to differentiate “creator” vs. “consumer”:

1. **Ways to Assign Roles**:  
   1. **B2C Custom Attributes**: You can store a user’s role in a custom attribute on their profile.  
   2. **API Database**: Your API keeps a “Users” table. Once the user signs in with B2C, you look up their account in your database to see if they’re a “creator” or “consumer.”  

2. **Enforce Roles in the API**  
   - In your `passport` callback, you get user info from the token (like `req.user`).  
   - Then, you can cross-reference that user in your own database if you want more details:
     ```js
     app.post('/api/upload', 
       passport.authenticate('oauth-bearer', { session: false }),
       async (req, res) => {
         // Suppose we get the user’s "sub" (B2C’s unique ID) from the token
         const userId = req.user.sub;

         // Query your DB for that user
         const userRecord = await getUserByB2CId(userId); 
         if (userRecord.role !== 'creator') {
           return res.status(403).send('Forbidden: You are not a creator.');
         }

         // If role == 'creator', continue with file upload logic
         // ...
       }
     );
     ```
   - For simpler use cases, you could store the role directly in the B2C token with a custom claim, then check `req.user.extension_role`.

---

## Part F: **Deploy and Test**

1. **Deploy Your Updated Back-End** to Azure App Service (just as before).  
2. **Set Environment Variables** for your B2C config in the Azure Portal → your App Service → **Configuration** → **Application settings**. For example:
   - `TENANT_NAME`, `CLIENT_ID`, `POLICY_NAME`, etc.
3. **Restart the App Service** to pick up the new environment variables.  
4. **Try Logging In** from a front-end or Postman:
   - Acquire a token from `https://<your-tenant-name>.b2clogin.com/<your-tenant-name>.onmicrosoft.com/oauth2/v2.0/authorize?p=<your-policy>` (this is typically handled by the front-end redirect).  
   - Use that token on your protected endpoints.
5. **Verify Role Behavior**:
   - If you’re storing roles in your DB, make sure at least one user is flagged as “creator.”  
   - Test that “consumer” role can’t perform restricted actions (like uploading).

---

## Common Tips & Troubleshooting

1. **JWT Expiration**: Tokens typically expire after a certain time. Your front-end should handle refreshing.  
2. **Insufficient Claims**: If you need custom claims (like roles), set them up in **User Attributes** or your database.  
3. **Policy Name**: Double-check you’re using the correct policy name for the user flow.  
4. **Redirect URIs**: If your front-end fails to redirect, ensure the **reply URL** matches exactly in the Azure AD B2C App Registration settings.  
5. **Logging**: Turn on logging or check the **Log Stream** in Azure to see authentication errors.

---

### Final Recap

- **Azure AD B2C** allows easy user sign-up/sign-in and token generation.  
- **Your API** needs to **validate** these tokens via a library (e.g., `passport-azure-ad`) or any JWT validation library.  
- **Roles** can be checked either from **custom attributes** in B2C or from your **own database** to ensure “creator” vs. “consumer” access.  

With authentication and authorization in place, you’ve added a critical security layer, allowing you to confidently move on to building your final **front-end** and user experiences.
