For the **Master Dashboard** in Vooglue, you'll need several APIs to fetch and update data dynamically instead of relying on dummy data. Here's the breakdown of the necessary APIs, categorized into **pull (GET)** and **push (POST/PUT/DELETE)** requests:

---

### **1. User & Account Information**
- **GET `/api/user/profile`** (Pull) → Fetches logged-in user's profile details, such as:
  - Name, account type, and selected roles.
  - Subscription level (**PRO, COL, ENT**).
  - Existing settings.

- **PUT `/api/user/profile`** (Push) → Updates user details if they change roles or preferences.

---

### **2. Artwork Management**
- **GET `/api/artworks`** (Pull) → Retrieves all artworks for the logged-in artist, including:
  - Artwork ID, title, description, status (For Sale, Sold, Not For Sale, etc.).
  - Images, pricing details, and ownership data.

- **POST `/api/artworks`** (Push) → Adds a new artwork when a user submits via `AddArtModal.js`.

- **PUT `/api/artworks/:id`** (Push) → Updates an artwork’s details (title, price, sale status, etc.).

- **DELETE `/api/artworks/:id`** (Push) → Deletes an artwork.

---

### **3. Venue Matching & Location Services**
- **GET `/api/venues?location={userLocation}`** (Pull) → Fetches **nearby art display venues**:
  - Venue name, address, contact info, distance, and rating.
  - Only shows venues that accept new artists.

- **POST `/api/venues/notify`** (Push) → Sends a request to notify users when new venues sign up near them.

---

### **4. Subscription & Upgrade**
- **GET `/api/subscription/plans`** (Pull) → Retrieves available subscription plans & pricing.

- **POST `/api/subscription/upgrade`** (Push) → Processes a **subscription upgrade** when a user chooses a plan.

- **GET `/api/subscription/status`** (Pull) → Fetches the user’s **current subscription status**.

---

### **5. Pie Chart Data (Artwork Stats)**
- **GET `/api/artworks/stats`** (Pull) → Returns aggregated statistics for `ArtPieChart.js`:
  - Number of artworks in each status (For Sale, Sold, On Hold, etc.).

---

### **6. AI Assistance Features**
If the AI-generated descriptions and pricing tools are server-side:
- **POST `/api/ai/generate-description`** (Push) → Sends artwork details & receives an AI-generated description.
- **POST `/api/ai/price-suggestion`** (Push) → Sends artwork details & gets an AI-based pricing suggestion.

---

### **7. Notifications & Alerts**
- **GET `/api/notifications`** (Pull) → Fetches any **alerts or messages** for the user (e.g., artwork sold, new venues joined).
- **POST `/api/notifications/subscribe`** (Push) → Subscribes the user to **venue alerts**.

---

### **Final API List**
| API Endpoint | Method | Purpose |
|-------------|--------|---------|
| `/api/user/profile` | **GET** | Fetches user details (name, roles, subscription). |
| `/api/user/profile` | **PUT** | Updates user details. |
| `/api/artworks` | **GET** | Retrieves the user's artworks. |
| `/api/artworks` | **POST** | Adds a new artwork. |
| `/api/artworks/:id` | **PUT** | Updates an artwork's details. |
| `/api/artworks/:id` | **DELETE** | Deletes an artwork. |
| `/api/venues?location={userLocation}` | **GET** | Fetches nearby art display venues. |
| `/api/venues/notify` | **POST** | Subscribes user to venue notifications. |
| `/api/subscription/plans` | **GET** | Fetches available subscription plans. |
| `/api/subscription/upgrade` | **POST** | Upgrades user’s subscription. |
| `/api/subscription/status` | **GET** | Checks the user's current subscription. |
| `/api/artworks/stats` | **GET** | Retrieves artwork status stats for the pie chart. |
| `/api/ai/generate-description` | **POST** | Generates an AI-based artwork description. |
| `/api/ai/price-suggestion` | **POST** | Provides an AI-based artwork price suggestion. |
| `/api/notifications` | **GET** | Fetches user notifications (sales, venue alerts). |
| `/api/notifications/subscribe` | **POST** | Subscribes user to notifications. |

---

This covers **all the necessary APIs** you need for the **Master Dashboard**.  
Let me know if you want to refine this list further!