# Building the Front-end

Below is a **step-by-step, guide** for building the front-end (both **Consumer** and **Creator** views) for your cloud-native photo/video sharing application. The goal is to keep it as straightforward as possible, using **basic HTML, CSS, and JavaScript**—so you don’t need a deep technical background or complicated frameworks unless you want them. The steps and code are only a guide and not a direct and complete solution. 

---

## Overview of Front-End Tasks

1. **Decide on Your Approach**  
   - **Option A: Basic HTML + JavaScript**. Easiest to grasp for beginners.  
   - **Option B: Use a Front-End Framework (Optional)** – like React, Vue, or Angular. This might be more advanced than necessary for a simple project.

2. **Create Consumer Pages**  
   - **Home/Search Page**: Display and filter media items.  
   - **Media Detail Page**: Show a single media item, plus comments/ratings.

3. **Create Creator Pages**  
   - **Upload Page**: Form for uploading new media (title, caption, file).  
   - **Manage Page** (optional): List existing uploads for editing or deleting.

4. **Authentication Integration**  
   - Make sure “login” and “logout” flows from Step 5 (Azure AD B2C) connect with your front-end.  
   - Store the user’s JWT (token) and use it to call your protected API endpoints.

5. **Call the REST API**  
   - Use `fetch()` or `axios` to interact with your back-end: get media lists, post comments, etc.

6. **Deploy Front-End**  
   - Host your static files on **Azure Storage Static Web Hosting** or in the same **Azure App Service**.

---

## Part A: **Set Up Your Project Structure**

1. **Create a Folder**  
   - Example: `my-media-frontend/`.
2. **Files Within**: 
   - `index.html` – your home/search page (consumer view).
   - `upload.html` – your creator upload page.
   - `styles.css` – your main CSS (optional, or inline CSS).
   - `app.js` – your main JavaScript for calling the API, handling logins, etc.
   - Additional `.html` files as needed (e.g., `media-detail.html`).

**Directory Example**:
```
my-media-frontend
|-- index.html
|-- upload.html
|-- media-detail.html
|-- styles.css
|-- app.js
```

---

## Part B: **Consumer View**

### 1. **Home/Search Page (`index.html`)**

**Goal**: Display a list of media items with a simple search/filter feature.

1. **HTML Structure** (simple example):
   ```html
   <!DOCTYPE html>
   <html>
   <head>
     <meta charset="UTF-8" />
     <title>My Media App - Home</title>
     <link rel="stylesheet" href="styles.css" />
   </head>
   <body>
     <h1>My Media App</h1>

     <!-- Navigation / Login Info -->
     <div id="user-info">
       <!-- We'll dynamically show user status (e.g., logged in or not) -->
       <button id="loginBtn">Login</button>
       <button id="logoutBtn" style="display:none;">Logout</button>
     </div>

     <!-- Search Bar -->
     <div>
       <input type="text" id="searchInput" placeholder="Search media by title...">
       <button id="searchBtn">Search</button>
     </div>

     <!-- Media List -->
     <div id="media-list"></div>

     <script src="app.js"></script>
   </body>
   </html>
   ```
2. **JavaScript (in `app.js`)**:
   ```js
   // Example: fetch and display media
   const apiBaseUrl = 'https://<your-api-name>.azurewebsites.net/api'; // Replace with your actual API URL

   // Event listeners
   document.getElementById('searchBtn').addEventListener('click', loadMedia);
   document.getElementById('loginBtn').addEventListener('click', login);
   document.getElementById('logoutBtn').addEventListener('click', logout);

   async function loadMedia() {
     const query = document.getElementById('searchInput').value || '';
     // For simplicity, let's say your API supports a query param like ?search=...
     const response = await fetch(`${apiBaseUrl}/media?search=${encodeURIComponent(query)}`);
     const mediaItems = await response.json();

     const mediaListDiv = document.getElementById('media-list');
     mediaListDiv.innerHTML = ''; // Clear old results

     mediaItems.forEach(item => {
       const div = document.createElement('div');
       div.innerHTML = `
         <h3>${item.title}</h3>
         <p>${item.caption}</p>
         <button onclick="viewMediaDetail(${item.mediaId})">View Details</button>
       `;
       mediaListDiv.appendChild(div);
     });
   }

   // Navigating to detail page
   function viewMediaDetail(mediaId) {
     // We can do a simple window location approach
     window.location.href = `media-detail.html?mediaId=${mediaId}`;
   }

   // Placeholder login/logout
   function login() {
     // This should redirect to your Azure AD B2C sign-in page or handle sign-in logic
     alert('Login flow not implemented in this basic example!');
   }
   function logout() {
     // Clear token, redirect as needed
   }

   // Load media on page load
   loadMedia();
   ```
3. **Styling** (`styles.css`) – purely optional. Keep it simple or style as desired.

---

### 2. **Media Detail Page (`media-detail.html`)**

**Goal**: Show a single media item, plus comments and ratings.

1. **HTML Structure**:
   ```html
   <!DOCTYPE html>
   <html>
   <head>
     <meta charset="UTF-8" />
     <title>Media Detail</title>
     <link rel="stylesheet" href="styles.css" />
   </head>
   <body>
     <h1>Media Detail</h1>

     <div id="media-container"></div>

     <!-- Comment Section -->
     <div id="comment-section">
       <h2>Comments</h2>
       <div id="comments-list"></div>
       <textarea id="commentInput" placeholder="Write a comment..."></textarea>
       <button id="commentBtn">Post Comment</button>
     </div>

     <script src="app.js"></script>
     <script>
       // On page load, parse the mediaId from URL and fetch the data
       const urlParams = new URLSearchParams(window.location.search);
       const mediaId = urlParams.get('mediaId');
       loadMediaDetail(mediaId);

       document.getElementById('commentBtn').addEventListener('click', () => postComment(mediaId));
     </script>
   </body>
   </html>
   ```
2. **Functions in `app.js`** (extending the previous example):
   ```js
   async function loadMediaDetail(mediaId) {
     // Fetch the single media item
     const response = await fetch(`${apiBaseUrl}/media/${mediaId}`);
     const mediaItem = await response.json();

     // Display media info
     const container = document.getElementById('media-container');
     container.innerHTML = `
       <h3>${mediaItem.title}</h3>
       <p>${mediaItem.caption}</p>
       <!-- If you have an image/video URL, embed it here -->
       ${mediaItem.fileUrl ? `<img src="${mediaItem.fileUrl}" alt="${mediaItem.title}" />` : ''}
     `;

     // Load comments
     loadComments(mediaId);
   }

   async function loadComments(mediaId) {
     const response = await fetch(`${apiBaseUrl}/media/${mediaId}/comments`);
     const comments = await response.json();
     const commentsList = document.getElementById('comments-list');
     commentsList.innerHTML = '';
     comments.forEach(c => {
       const commentDiv = document.createElement('div');
       commentDiv.innerHTML = `<p>${c.userName}: ${c.commentText}</p>`;
       commentsList.appendChild(commentDiv);
     });
   }

   async function postComment(mediaId) {
     const commentText = document.getElementById('commentInput').value;
     // If your API needs a bearer token:
     const token = localStorage.getItem('authToken'); // e.g., from B2C
     const response = await fetch(`${apiBaseUrl}/media/${mediaId}/comments`, {
       method: 'POST',
       headers: {
         'Content-Type': 'application/json',
         'Authorization': `Bearer ${token}`
       },
       body: JSON.stringify({ commentText })
     });

     if (response.ok) {
       document.getElementById('commentInput').value = '';
       loadComments(mediaId); // Refresh
     } else {
       alert('Error posting comment');
     }
   }
   ```
3. **Ratings** can be handled similarly (e.g., a `POST /media/:id/rating` endpoint).

---

## Part C: **Creator View**

### 1. **Upload Page (`upload.html`)**

**Goal**: Let “creator” users upload new media files, along with metadata (title, caption, etc.).

1. **HTML Structure**:
   ```html
   <!DOCTYPE html>
   <html>
   <head>
     <meta charset="UTF-8" />
     <title>Upload Media</title>
     <link rel="stylesheet" href="styles.css" />
   </head>
   <body>
     <h1>Upload Media</h1>

     <form id="uploadForm">
       <label>Title</label>
       <input type="text" id="titleInput" required>

       <label>Caption</label>
       <textarea id="captionInput"></textarea>

       <label>Select File</label>
       <input type="file" id="fileInput" accept="image/*,video/*" required>

       <button type="submit">Upload</button>
     </form>

     <script src="app.js"></script>
     <script>
       document.getElementById('uploadForm').addEventListener('submit', uploadMedia);
     </script>
   </body>
   </html>
   ```
2. **Upload Function** (`app.js`):
   ```js
   async function uploadMedia(event) {
     event.preventDefault(); // prevent form from reloading the page

     const title = document.getElementById('titleInput').value;
     const caption = document.getElementById('captionInput').value;
     const file = document.getElementById('fileInput').files[0];

     // Prepare form data
     const formData = new FormData();
     formData.append('mediaFile', file);
     formData.append('title', title);
     formData.append('caption', caption);

     // If your endpoint is protected (only creators can upload):
     const token = localStorage.getItem('authToken');

     const response = await fetch(`${apiBaseUrl}/upload`, {
       method: 'POST',
       headers: {
         'Authorization': `Bearer ${token}`
       },
       body: formData
     });

     if (response.ok) {
       alert('Upload successful!');
       // Optionally redirect somewhere
     } else {
       alert('Upload failed.');
     }
   }
   ```
3. **Role Restriction**: If the user is not a “creator,” your back-end should return a `403 Forbidden`. You can also add client-side checks if you store user roles locally.

---

## Part D: **Handling Authentication in the Front-End**

1. **When a User Logs In** (from Step 5 with Azure AD B2C):
   - Typically, your front-end redirects to **B2C sign-in** page. B2C then returns the user with a **JWT token**.  
   - You store that token in **`localStorage`** or **`sessionStorage`**:
     ```js
     // Pseudo-code after successful login:
     localStorage.setItem('authToken', token);
     localStorage.setItem('userRole', 'creator'); // or 'consumer' if known
     ```
2. **Check If Logged In**:
   - On any page, you can check:
     ```js
     const token = localStorage.getItem('authToken');
     if (!token) {
       // user is not logged in, show "login" button
     } else {
       // user is logged in, show "logout" button, etc.
     }
     ```
3. **Logout**:
   - Simply remove the token:
     ```js
     function logout() {
       localStorage.removeItem('authToken');
       localStorage.removeItem('userRole');
       window.location.reload(); // or redirect
     }
     ```
4. **Role-Based UI**:  
   - If you store the user’s role (creator/consumer) in localStorage, you can hide or show certain pages or buttons:
     ```js
     const role = localStorage.getItem('userRole');
     if (role === 'creator') {
       // show upload page link
     }
     ```

---

## Part E: **Testing Locally**

1. **Open `index.html`** in your browser.  
2. **Check** that the media list loads from your local or deployed API.  
3. **Navigate** to `upload.html` (if you’re a creator). Attempt an upload to confirm it reaches Azure Blob Storage via the API.  
4. **Try** leaving a comment on `media-detail.html`.  

---

## Part F: **Deploying the Front-End**

You have **two main options** for hosting static content:

### Option 1: **Azure Static Web Apps** (Recommended for Simplicity)
1. **Go to** Azure Portal → search for **“Static Web Apps”** → **+ Create**.  
2. **Point** it to your GitHub repository or choose the “Other” option for manual deployment.  
3. **Build/Release Configuration** can be minimal for a pure HTML/JS site (no build step).  
4. **Result**: You’ll get a URL like `https://<random-name>.zwp.azurestaticapps.net` hosting your front-end.  

### Option 2: **Deploy to the Same App Service as Your API**
1. **Zip Deploy** or “Deploy from GitHub” to your **Azure App Service**.  
2. **Put** your `.html`, `.css`, and `.js` files in a folder.  
3. **Configure** Express (if you’re using Node) to serve static files:
   ```js
   app.use(express.static('public')); // if your HTML/CSS/JS are in 'public' folder
   ```
4. **Access** your site via `https://<your-app>.azurewebsites.net/index.html`.

Either way, your front-end and API can communicate as long as you update the `apiBaseUrl` to the correct endpoint.

---

## Part G: **Next Steps**

1. **Improve Styling**: Use CSS frameworks like Bootstrap if desired.  
2. **Enhance the Search**: Full-text search, filtering by location, date, etc.  
3. **Add More Creator Tools**: Editing or deleting existing media.  
4. **Add More Consumer Features**: Rating system, social sharing, etc.  
5. **Optimize**: Implement caching, lazy loading for images, etc.

---

## Final Recap

By following these steps, you’ll have:

1. **Consumer Pages**:
   - A **Home/Search Page** listing media items (from the API).  
   - A **Media Detail Page** showing comments, allowing new comments or ratings.

2. **Creator Pages**:
   - An **Upload Page** enabling new media uploads (files + metadata).

3. **Authentication Integration**:
   - A way for users to log in via Azure AD B2C (Step 5), store a token, and call secured endpoints.

4. **Deployment**:
   - A live website on Azure (either via **Static Web Apps** or your **App Service**).

Congratulations! You now have a front-end that interacts with your REST API, storing data in Azure’s database and media in Blob Storage. 
