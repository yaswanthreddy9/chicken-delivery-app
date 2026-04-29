https://yaswanthreddy9.github.io/Quick-delivery-app/

# 🛒 Customer Ordering App (User Frontend)

**Stack:** HTML5, CSS3, Vanilla Javascript, Capacitor, Firebase (Firestore)
**Architecture:** Pure Web-Wrapper App (No Native Code Required)
**Role:** The entry point of the ecosystem. Customers browse, checkout, and push data to Firestore, which triggers the Admin Dashboard and Driver native alerts.

## 📖 Overview
Unlike the Driver App—which requires native Android permissions to bypass battery savers and sound alarms—the Customer App is a standard, polite application. It relies entirely on standard web technologies wrapped in Capacitor. 

Its primary function is to construct a JSON object (the "Order") and push it to the `orders` collection in Firebase. It then actively listens to that document to provide real-time status updates to the customer.

---

## 🏗️ System Architecture & Data Flow

### 1. The Storefront (UI Layer)
* **Tech:** Standard HTML/CSS.
* **Function:** Displays items dynamically (can be hardcoded initially or fetched from a `products` collection in Firestore).
* **Interaction:** Users click "Add to Cart," which fires Javascript functions to update the local state.

### 2. State Management (Cart Logic)
* **Tech:** Javascript Arrays/Objects.
* **Function:** Maintains the current `cart` array. Calculates subtotal, applies tax/delivery fees, and handles quantity adjustments.
* **Local Storage:** Uses `window.localStorage` to save the cart so if the user closes the app accidentally, they don't lose their items.

### 3. The Firebase Hand-off (Checkout)
* **Trigger:** User clicks "Place Order".
* **Action:** Javascript compiles the cart array, user ID, total price, and delivery address into a single JSON object.
* **Database Write:** Calls `db.collection('orders').add(orderObject)`.
* **Ecosystem Reaction:** This single write operation instantly appears on the Admin Dashboard. Once the Admin assigns it, the Cloud Function wakes up the Driver App.

### 4. Real-Time Tracking (The Listener)
* **Tech:** Firebase `onSnapshot()`.
* **Function:** Once the order is placed, the app switches to a Tracking Screen. It actively listens to its specific document ID in the `orders` collection.
* **Reactivity:** When the Admin or Driver updates the `status` field (e.g., from `Pending` -> `Out for Delivery`), the Customer App UI updates instantly without requiring a page refresh.

---

## 📂 Recommended Folder Structure

```text
UserApp/
│
├── index.html          # Main storefront UI and Cart modal
├── tracking.html       # The live order status screen
├── css/
│   └── style.css       # All visual styling
└── js/
    ├── firebase.js     # Firebase config and initialization
    ├── cart.js         # Cart array, addition/removal logic, and local storage
    └── checkout.js     # Pushing the final order to Firestore and initiating tracking
```

---

## 🛠️ Step-by-Step Developer Implementation Guide

### Step 1: Initialize the Project
Create the basic web folder and initialize Capacitor to wrap it for mobile.
```bash
npm install @capacitor/core @capacitor/cli
npx cap init "User App" "com.delivery.user"
npm install @capacitor/android
npx cap add android
```

### Step 2: Build the Frontend (index.html)
Focus purely on the web experience first. Build out the item cards and the cart interface. Do not connect Firebase yet. Ensure the Javascript can successfully push items into a `cart` array and calculate a total.

### Step 3: Connect Firebase (checkout.js)
Link your existing Firebase project. When the user checks out, structure the data exactly like this so the Admin and Driver apps can read it perfectly:

```javascript
const orderPayload = {
    userId: "customer_123",
    customerName: "John Doe",
    customerAddress: "123 Main St, Apt 4B",
    items: cartArray,
    totalAmount: 45.99,
    status: "Pending", // Crucial: Starts as Pending
    driverId: null,    // Admin will fill this in later
    timestamp: firebase.firestore.FieldValue.serverTimestamp()
};

db.collection("orders").add(orderPayload).then((docRef) => {
    console.log("Order placed successfully with ID: ", docRef.id);
    // Redirect user to tracking screen
});
```

### Step 4: Implement Tracking
On the tracking screen, set up the listener to watch the order.

```javascript
db.collection("orders").doc(orderId)
    .onSnapshot((doc) => {
        const orderData = doc.data();
        document.getElementById("status-text").innerText = `Status: ${orderData.status}`;
        
        // Example: If status becomes 'Delivered', show a celebration animation
        if(orderData.status === "Delivered") {
            showOrderCompleteUI();
        }
    });
```

---

## 🚀 Deployment Checklist
- [ ] Connect existing Firebase Config keys to `firebase.js`.
- [ ] Ensure Firestore Security Rules allow authenticated users to write to `orders`.
- [ ] Run `npx cap sync`.
- [ ] Open in Android Studio to build the final APK.
