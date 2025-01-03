# Chrome-Extension-for-Monitoring-a-Website
We have a client that uses an ecommerce website to run their business.  When web orders occur, there is a form field (configurable) on a specific page/URL (configurable) that is updated when the page is refreshed to indicate the number of pending web orders.  We would like a browser extension that would refresh the page at intervals (configurable, likely 1 minute) to update the number of pending web orders.

In addition to refreshing, we would like the extension to perform actions when the number is greater than zero and again when it is back to zero.  We have found a USB light that can be controlled by URL.  So there is one URL to turn on the light and another to turn off the light.  When the number that is on the monitored page is greater than zero, we would like to call the URL (configurable) to turn on the light.  When the number is zero, we would call the URL (configurable) to turn off the light.

We would want all URLs, intervals, form field ID,  etc to be configurable on the extension's properties page.  We will want the developer to work with us to get the app approved in the  Google Store and will likely look to sell it in the future.
--------
Creating a Chrome extension for monitoring pending web orders and controlling a USB light based on those orders can be broken down into several core tasks. Here’s a detailed plan and code structure to help you build this extension, which will include:

    Refreshing the page at intervals (configurable).
    Monitoring the number of pending web orders.
    Turning on/off the light based on the number of orders.
    Making all URLs, intervals, and form field IDs configurable.

We’ll structure the extension with the following components:

    Manifest File: Defines the permissions and the background settings.
    Popup (configurable settings): A simple interface for users to input the settings (e.g., URLs, intervals).
    Content Script: Monitors the page and performs the refresh and checking actions.
    Background Script: Handles the light control and URL calls when the number of pending orders changes.

1. Manifest File (manifest.json)

This file will configure the extension’s permissions, scripts, and popup.

{
  "manifest_version": 3,
  "name": "Order Monitor with Light Control",
  "description": "Monitor pending orders and control a light based on the order status.",
  "version": "1.0",
  "permissions": [
    "activeTab",
    "storage",
    "alarms"
  ],
  "background": {
    "service_worker": "background.js"
  },
  "action": {
    "default_popup": "popup.html"
  },
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
      "js": ["content.js"]
    }
  ],
  "host_permissions": [
    "http://*/*",
    "https://*/*"
  ]
}

2. Popup Interface (popup.html)

This file provides a simple interface to configure the extension, such as the monitored URL, form field ID, light control URLs, and refresh interval.

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Order Monitor Settings</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      width: 300px;
      padding: 10px;
    }
    input, button {
      width: 100%;
      margin-bottom: 10px;
      padding: 8px;
    }
  </style>
</head>
<body>
  <h2>Order Monitor Settings</h2>
  <label for="url">Monitored Page URL:</label>
  <input type="text" id="url" placeholder="Enter URL">
  
  <label for="formFieldId">Form Field ID:</label>
  <input type="text" id="formFieldId" placeholder="Enter Form Field ID">
  
  <label for="lightOnUrl">Light On URL:</label>
  <input type="text" id="lightOnUrl" placeholder="Enter Light On URL">
  
  <label for="lightOffUrl">Light Off URL:</label>
  <input type="text" id="lightOffUrl" placeholder="Enter Light Off URL">
  
  <label for="refreshInterval">Refresh Interval (seconds):</label>
  <input type="number" id="refreshInterval" value="60">
  
  <button id="saveSettings">Save Settings</button>

  <script src="popup.js"></script>
</body>
</html>

3. Popup Script (popup.js)

The JavaScript here handles storing user input settings in Chrome's local storage.

document.getElementById('saveSettings').addEventListener('click', () => {
  const url = document.getElementById('url').value;
  const formFieldId = document.getElementById('formFieldId').value;
  const lightOnUrl = document.getElementById('lightOnUrl').value;
  const lightOffUrl = document.getElementById('lightOffUrl').value;
  const refreshInterval = parseInt(document.getElementById('refreshInterval').value, 10);
  
  // Save settings to localStorage
  chrome.storage.local.set({
    monitoredUrl: url,
    formFieldId: formFieldId,
    lightOnUrl: lightOnUrl,
    lightOffUrl: lightOffUrl,
    refreshInterval: refreshInterval
  }, function() {
    alert('Settings saved successfully');
  });
});

4. Content Script (content.js)

This script will monitor the web page for the pending orders count by checking the form field at regular intervals.

// Retrieve the saved settings
chrome.storage.local.get(['monitoredUrl', 'formFieldId', 'lightOnUrl', 'lightOffUrl'], (result) => {
  const formFieldId = result.formFieldId;
  
  // Function to check the pending orders count and trigger light actions
  function checkPendingOrders() {
    const pendingOrders = document.getElementById(formFieldId).innerText;

    if (parseInt(pendingOrders, 10) > 0) {
      // Turn on light if orders are pending
      fetch(result.lightOnUrl)
        .then(response => console.log("Light turned on"))
        .catch(err => console.error("Error turning on light", err));
    } else {
      // Turn off light if no orders are pending
      fetch(result.lightOffUrl)
        .then(response => console.log("Light turned off"))
        .catch(err => console.error("Error turning off light", err));
    }
  }

  // Refresh the page at the set interval
  setInterval(checkPendingOrders, result.refreshInterval * 1000);
});

5. Background Script (background.js)

This script will handle background tasks like refreshing the page at set intervals and checking for pending orders.

chrome.runtime.onInstalled.addListener(() => {
  console.log("Order Monitor Extension Installed");
});

6. Testing and Running the Extension

    Go to chrome://extensions/ in your browser.
    Enable "Developer mode" in the top right corner.
    Click "Load unpacked" and select the folder containing your extension files.
    Once installed, click on the extension icon, input the settings (URL, form field ID, light URLs, etc.).
    Ensure the form field is correctly identified and the light URLs are correct. After saving settings, the extension will periodically check for pending orders and turn the light on/off accordingly.

7. Conclusion

With this extension, users can easily monitor the number of pending web orders and control an external light based on that number. The extension provides configurable options for the monitored URL, form field ID, light control URLs, and refresh interval. All settings are stored in Chrome's local storage, and the extension automatically checks and updates the order status at user-defined intervals.

This is a functional MVP, and you can extend it by adding additional features like error handling, better UI, or integrating with other APIs.
