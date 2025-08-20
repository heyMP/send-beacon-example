# Send Beacon API Demo

This project is a fun, interactive landing page that demonstrates how to use the Beacon API in conjunction with the `beforeunload` and `visibilitychange` events to reliably send analytics data when a user leaves a page.

## The Purpose

In web development, it's often crucial to know when a user navigates away from your site. This allows you to save their session, log analytics, or perform other cleanup tasks. However, this can be tricky because modern browsers might not reliably fire older events like `unload`, and you can't run asynchronous requests (like `fetch`) during the page unload process.

This demo showcases a modern, robust solution using a combination of two key events:

1.  **`beforeunload`**: A long-standing event that signals a user is about to leave the page. We use this as our primary indicator that the session is ending.
2.  **`visibilitychange`**: A more modern and reliable event that tells us when the page is no longer visible to the user (e.g., they switched tabs or minimized the browser). This is great for tracking when a user is temporarily away, but might return.

To send the final piece of data, we use the **Beacon API**. This API is specifically designed to send small amounts of data from the browser to a web server *without* delaying the unload of the document or affecting the performance of the next navigation.

## The Code

Here is the core JavaScript code that powers this demonstration. It sends a beacon when the page first loads, when its visibility changes, and when the user leaves the page entirely.

```javascript
let unloaded = false;
const sendBeacon = (type) => {
    const data = JSON.stringify({
        [`session_${type}`]: Date.now()
    });
    navigator.sendBeacon('http://localhost:3000/', data);
}

sendBeacon('start');
window.addEventListener('beforeunload', () => {
    unloaded = true;
    sendBeacon('end');
});
document.addEventListener('visibilitychange', function () {
    if (unloaded) {
        return;
    }
    if (document.visibilityState === 'hidden') {
        sendBeacon('hidden');
    } else {
        sendBeacon('restored');
    }
});
```

## How to Run Locally

To run this demo on your own machine, follow these simple steps:

1.  **Clone the repository (if you haven't already):**
    ```bash
    git clone <repository-url>
    cd send-beacon-test
    ```

2.  **Start the server:**
    Open your terminal and run the following command:
    ```bash
    npm run start
    ```
    You should see a message confirming that the server is running:
    ```
    Server running at http://localhost:3000/
    ```

3.  **Open the page in your browser:**
    Navigate to `http://localhost:3000/` in your web browser.

4.  **Test it out!**
    -   Navigate to a different website.
    -   Close the browser tab.
    -   Switch to another tab and then back again.

    After each of these actions, check your terminal. You'll see the beacon messages being logged by the Node.js server, demonstrating that the API calls were successful!
