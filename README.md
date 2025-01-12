# TASK-4-CHROME-EXTENSION-FOR-TIME-TRACKING-AND-PRODUCTIVITY-ANALYTICS
CODETECH Internship 

// Chrome Extension: Productivity Tracker

// Manifest.json
{
  "manifest_version": 3,
  "name": "Productivity Tracker",
  "version": "1.0.0",
  "description": "Track the time spent on websites and analyze productivity.",
  "permissions": ["tabs", "storage", "activeTab", "alarms"],
  "background": {
    "service_worker": "background.js"
  },
  "action": {
    "default_popup": "popup.html",
    "default_icon": "icon.png"
  },
  "icons": {
    "48": "icon.png",
    "128": "icon-large.png"
  }
}

// background.js
let siteTimes = {};
let currentSite = null;
let startTime = null;

// Detect active tab changes
chrome.tabs.onActivated.addListener(async (activeInfo) => {
  await updateCurrentTabTime();
  const tab = await chrome.tabs.get(activeInfo.tabId);
  trackSite(tab.url);
});

// Detect URL changes
chrome.tabs.onUpdated.addListener((tabId, changeInfo, tab) => {
  if (changeInfo.status === 'complete' && tab.active) {
    trackSite(tab.url);
  }
});

// Track time spent on a site
function trackSite(url) {
  if (!url) return;
  const domain = new URL(url).hostname;

  if (currentSite) {
    const timeSpent = Date.now() - startTime;
    siteTimes[currentSite] = (siteTimes[currentSite] || 0) + timeSpent;
  }

  currentSite = domain;
  startTime = Date.now();
}

// Update current tab's time when the user switches tabs or closes the browser
async function updateCurrentTabTime() {
  if (currentSite && startTime) {
    const timeSpent = Date.now() - startTime;
    siteTimes[currentSite] = (siteTimes[currentSite] || 0) + timeSpent;
  }
  startTime = Date.now();
}

// Store data periodically
chrome.alarms.create('saveData', { periodInMinutes: 1 });
chrome.alarms.onAlarm.addListener((alarm) => {
  if (alarm.name === 'saveData') {
    chrome.storage.local.set({ siteTimes });
  }
});

// popup.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Productivity Tracker</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 10px;
    }
    h1 {
      font-size: 18px;
      margin-bottom: 10px;
    }
    table {
      width: 100%;
      border-collapse: collapse;
    }
    th, td {
      text-align: left;
      padding: 8px;
    }
    th {
      background-color: #f4f4f4;
    }
    tr:nth-child(even) {
      background-color: #f9f9f9;
    }
  </style>
</head>
<body>
  <h1>Productivity Tracker</h1>
  <table>
    <thead>
      <tr>
        <th>Website</th>
        <th>Time Spent (mins)</th>
      </tr>
    </thead>
    <tbody id="siteTimes"></tbody>
  </table>
  <button id="reset">Reset Data</button>
  <script src="popup.js"></script>
</body>
</html>

// popup.js
const tableBody = document.getElementById('siteTimes');
const resetButton = document.getElementById('reset');

function displaySiteTimes(siteTimes) {
  tableBody.innerHTML = '';
  for (const [site, time] of Object.entries(siteTimes)) {
    const row = document.createElement('tr');
    row.innerHTML = `
      <td>${site}</td>
      <td>${(time / 60000).toFixed(2)}</td>
    `;
    tableBody.appendChild(row);
  }
}

resetButton.addEventListener('click', () => {
  chrome.storage.local.clear(() => {
    tableBody.innerHTML = '';
  });
});

chrome.storage.local.get('siteTimes', (data) => {
  displaySiteTimes(data.siteTimes || {});
});
