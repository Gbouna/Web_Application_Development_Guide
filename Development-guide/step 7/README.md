# Scalability

Below is a **step-by-step, guide** for adding **scalability** and exploring **additional enhancements** in your cloud-native photo/video sharing application. This step is all about **making sure your app can handle growing traffic** and adding extra features if you have the time and resources.

---

## Overview of Scalability & Enhancements

1. **Scalability Basics**
   - Adjusting your App Service plan or database tier.
   - Adding caching for performance.
   - Using Content Delivery Networks (CDNs) for faster media delivery.

2. **Optional Enhancements**
   - Media conversion/transcoding.
   - Advanced user engagement features (likes, follows, notifications).
   - Monitoring and logging (Application Insights).
   - Disaster recovery and backups.

---

## Part A: **Scalability Basics**

### 1. **Upgrade (or Downgrade) Your Service Plans**

- **App Service Plan**  
  1. Open the Azure Portal, find your **App Service** (the one hosting your API, and possibly your front-end).  
  2. Under **Settings**, select **Scale up (App Service plan)**.  
  3. Choose a higher tier (e.g., **B1** or **S1**) if you need more CPU, memory, or concurrent connections.  
  4. Apply changes. Azure will upgrade the underlying resources so your app can handle more traffic.

- **Azure SQL Database** or **Cosmos DB**  
  1. Go to your database in Azure.  
  2. For **SQL**, find **Compute + storage** in the left menu or in the Overview page. Click **Configure**.  
  3. Upgrade to a higher tier (e.g., from **Basic** to **Standard**). If you’re using **Cosmos DB**, adjust your **Request Units (RUs)** or **Throughput** to handle more reads/writes.  
  4. This ensures your database can handle higher load without performance issues.

> **Tip**: Always monitor **cost**. Higher tiers might exceed free credits. It’s best to move back down if usage falls.

---

### 2. **Caching for Better Performance**

- **What is Caching?**  
  Storing frequently accessed data in a fast, temporary location to reduce load on your database or storage.

- **Azure Cache for Redis**  
  1. Search for “Azure Cache for Redis” in the Portal → **+ Create**.  
  2. Choose a name, region, and a pricing tier (the **Basic** tier is the least expensive).  
  3. In your API code, whenever you fetch data from the database, you can first check the Redis cache. If data is there, return it immediately. If not, fetch from DB and store it in Redis.

- **Built-In Caching with App Service**  
  - If your front-end is static, it’s often automatically cached by the user’s browser. You can set caching headers in your server or storage.  
  - For example, in an **Express.js** app:
    ```js
    app.use(express.static('public', {
      maxAge: '1d' // Cache static assets for 1 day
    }));
    ```
  - This is a simple way to reduce repeated requests for the same files.

---

### 3. **Content Delivery Network (CDN)**

- **Why Use a CDN?**  
  - Distribute your media content (images/videos) across global edge servers, reducing latency for users far from your primary storage region.

- **Set Up an Azure CDN**  
  1. In the Azure Portal, search for **“CDN profiles”** → **+ Create**.  
  2. Choose **Azure CDN from Verizon** or **Azure CDN from Akamai** (both commonly used).  
  3. Select your storage account (where your media is stored) as the **origin**.  
  4. Once configured, you’ll get a CDN endpoint (e.g., `mycdnendpoint.azureedge.net`).  
  5. You can then replace your direct Blob Storage URL with the CDN URL for faster access.

> **Tip**: The free student tier might have limits on CDN usage, so keep an eye on potential costs.

---

## Part B: **Optional Enhancements**

### 1. **Media Conversion / Transcoding**

- **Azure Media Services** (AMS)  
  1. Search for **“Media Services”** in the Azure Portal → **+ Create** a Media Services account.  
  2. You can upload videos to AMS, then create **transforms** to automatically convert them to multiple resolutions or generate thumbnails.  
  3. This can be integrated into your back-end so that when a creator uploads a file, it’s passed to AMS for processing.  
  4. **Cost Alert**: AMS can get pricey if you process large amounts of video. Ensure you check the free tier or low-cost options.

- **Serverless Approach**  
  - For a simpler use case, you can run **Azure Functions** to create thumbnails for images automatically:
    1. Create an **Azure Function** that triggers on Blob upload.  
    2. When a new image is uploaded, the function scales it down to a thumbnail, then saves it back to Blob Storage.  
    3. Display these thumbnails on your front-end for faster load times.

---

### 2. **Advanced User Engagement**

- **Likes, Follows, Notifications**  
  - Add new database tables or containers to track likes or followers.  
  - Implement push notifications or email alerts using **Azure Communication Services** or **SendGrid** (for emails).

- **Social Sharing**  
  - Add “Share” buttons in your front-end that let users post links to popular social networks.  
  - Keep an eye on your app’s URL structure, making sure links go to the correct pages (e.g., `media-detail.html?mediaId=123`).

---

### 3. **Monitoring & Logging (Application Insights)**

- **Why Monitoring?**  
  - Monitor your application’s health, performance, and errors in real-time. 
- **Set Up**:  
  1. In the Portal, go to your **App Service** → **Application Insights** (on the left).  
  2. **Enable** Application Insights.  
  3. Optionally add an AI key to your code if you want more detailed logging.  
  4. Check the **Live Metrics** stream or the **Failures** section to see if users encounter errors.

---

### 4. **Disaster Recovery & Backups**

- **Regular Backups**  
  - For **Azure SQL**: You have automated backups, but you can also do manual **Export** to a BACPAC file or use the built-in backup feature.  
  - For **Cosmos DB**: It also handles backups automatically, but you can configure custom backup intervals.  
- **Geo-Replication**  
  - For mission-critical apps, replicate your DB to another region in case the primary region goes down.

---

## Part C: **Practical Example: Adding Redis Cache**

To make this concrete, here’s a quick how-to for adding an **Azure Cache for Redis** to your Node.js app:

1. **Create Azure Cache for Redis** in the portal:  
   - **Name**: `myMediaRedisCache`.  
   - **Pricing Tier**: Basic (C0 or C1).  
   - **Resource Group**: `MyMediaAppRG`.  
   - **Location**: same region as your App Service.

2. **Get Connection Settings**  
   - Under your Redis Cache, go to **“Access keys”**. You’ll see the **Host name** and **Primary key** (password).

3. **Install `redis` Package**  
   ```bash
   npm install redis
   ```
4. **Use It in Your Code**  
   ```js
   // db.js or cache.js
   const redis = require('redis');
   const client = redis.createClient({
     url: `redis://:<PASSWORD>@<HOSTNAME>:6380`,
     socket: {
       tls: true,
       rejectUnauthorized: false
     }
   });
   // Connect
   client.connect()
     .then(() => console.log('Connected to Redis'))
     .catch(console.error);

   module.exports = client;
   ```
5. **Cache a Database Query** (Example):
   ```js
   const cacheClient = require('./cache'); // your redis client

   app.get('/api/media', async (req, res) => {
     try {
       // Check cache first
       const cachedData = await cacheClient.get('mediaList');
       if (cachedData) {
         return res.json(JSON.parse(cachedData));
       }

       // If not in cache, query the database
       const pool = await sql.connect(process.env.SQL_CONNECTION_STRING);
       const result = await pool.request().query('SELECT * FROM Media');
       const mediaItems = result.recordset;

       // Store in cache for 60 seconds
       await cacheClient.setEx('mediaList', 60, JSON.stringify(mediaItems));

       res.json(mediaItems);
     } catch (error) {
       console.error(error);
       res.status(500).send('Error fetching media');
     }
   });
   ```
   - Now, subsequent requests to `GET /api/media` will serve from Redis cache if within 60 seconds, reducing DB load.

---

## Part D: **Testing Your Enhanced Setup**

1. **Load Testing**  
   - Use tools like [Apache JMeter](https://jmeter.apache.org/) or [Locust](https://locust.io/) to simulate multiple users hitting your API.  
   - See if the API remains responsive; if not, consider scaling up.

2. **Monitor Costs**  
   - In the Portal, go to **Cost Management** → **Cost analysis**.  
   - Set up **Budget alerts** if you want email notifications when your usage approaches a certain limit.

3. **Check Logs**  
   - If you added Application Insights, look at performance metrics (e.g., average response time, failed requests).

---
