---
title: "Running Apache Superset from Electron App"
description: "meta description"
date: 2024-07-26T05:00:00Z
image: "/images/posts/01.jpg"
categories: ["desktop", "superset"]
authors: ["Balaji Balasundaram"]
tags: ["electron", "desktop", "superset", "apache"]
draft: false
---

Installing and running Apache Superset using Docker is a convenient way to get started quickly. Here are the steps to set up Apache Superset using its official Docker image:

#### Step 1: Install Docker
First, ensure Docker is installed on your system. You can download and install Docker from here.

#### Step 2: Clone the Superset Repository
Clone the official Apache Superset repository, which contains the necessary Docker configurations.

```bash
git clone https://github.com/apache/superset.git
cd superset
```
#### Step 3: Set Up Environment Variables
Create a .env file in the root directory to define environment variables required by Docker Compose. You can use the example .env file provided in the repository:

```bash
cp docker/.env .env
```
#### Step 4: Initialize and Start Superset
Use Docker Compose to set up and start the Superset containers. This will create and start containers for Superset, Redis, and a PostgreSQL database.

```bash
docker-compose -f docker-compose-non-dev.yml pull
docker-compose -f docker-compose-non-dev.yml up -d
```
#### Step 5: Initialize the Superset Database
Once the containers are up and running, initialize the Superset database and create an admin user.

```bash
docker exec -it superset_app superset db upgrade
docker exec -it superset_app superset fab create-admin \
               --username admin \
               --firstname Superset \
               --lastname Admin \
               --email admin@superset.com \
               --password admin
docker exec -it superset_app superset init
```
#### Step 6: Access Superset
After completing the setup, you can access the Superset web UI by navigating to http://localhost:8088 in your web browser. Use the credentials you created for the admin user to log in.

#### Step 7: Custom Configuration (Optional)
You can customize your Superset configuration by editing the superset_config.py file. Create a custom configuration file and mount it in the Docker container by adding a volume in the docker-compose.yml file.

#### Step 8. Create the Electron App
If you dont have electron, install :

```bash
npm install -g electron
```

If you don't have an Electron app already, create one:

```bash
npm init electron-app my-electron-app
cd my-electron-app
npm install
```

#### Step 9. Configure Electron to Display Superset
Modify the main.js file to load your Apache Superset URL in a browser window:

```javascript
const { app, BrowserWindow } = require('electron');

function createWindow() {
  const mainWindow = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      nodeIntegration: true,
    },
  });

  // Load the Superset URL
  mainWindow.loadURL('http://localhost:8088'); // Change this to your Superset URL
}

app.on('ready', createWindow);

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    app.quit();
  }
});

app.on('activate', () => {
  if (BrowserWindow.getAllWindows().length === 0) {
    createWindow();
  }
});
```
#### Step 10. Handle Authentication
If your Superset instance requires authentication, you might need to handle it within the Electron app. One way to do this is by loading a login page first and then redirecting to the Superset dashboard upon successful login.

You can use IPC (Inter-Process Communication) to handle authentication:

Create a login window.
Capture the login credentials.
Send the credentials to the main process using IPC.
Perform the login in the main process and, if successful, load the Superset URL.
Example:

main.js

```javascript
const { app, BrowserWindow, ipcMain } = require('electron');
const fetch = require('node-fetch');

function createLoginWindow() {
  const loginWindow = new BrowserWindow({
    width: 400,
    height: 300,
    webPreferences: {
      nodeIntegration: true,
    },
  });

  loginWindow.loadFile('login.html');
}

function createMainWindow() {
  const mainWindow = new BrowserWindow({
    width: 800,
    height: 600,
    webPreferences: {
      nodeIntegration: true,
    },
  });

  mainWindow.loadURL('http://localhost:8088'); // Change this to your Superset URL
}

app.on('ready', createLoginWindow);

ipcMain.on('login', async (event, credentials) => {
  const response = await fetch('http://localhost:8088/login/', { // Adjust this URL to match your Superset login endpoint
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(credentials),
  });

  if (response.ok) {
    createMainWindow();
  } else {
    event.reply('login-failed');
  }
});

app.on('window-all-closed', () => {
  if (process.platform !== 'darwin') {
    app.quit();
  }
});

app.on('activate', () => {
  if (BrowserWindow.getAllWindows().length === 0) {
    createLoginWindow();
  }
});
```
login.html

```html
<!DOCTYPE html>
<html>
<head>
  <title>Login</title>
</head>
<body>
  <form id="login-form">
    <input type="text" id="username" placeholder="Username" required>
    <input type="password" id="password" placeholder="Password" required>
    <button type="submit">Login</button>
  </form>

  <script>
    const { ipcRenderer } = require('electron');

    document.getElementById('login-form').addEventListener('submit', (e) => {
      e.preventDefault();

      const credentials = {
        username: document.getElementById('username').value,
        password: document.getElementById('password').value,
      };

      ipcRenderer.send('login', credentials);
    });

    ipcRenderer.on('login-failed', () => {
      alert('Login failed. Please check your credentials.');
    });
  </script>
</body>
</html>
```
#### Step 11. Build and Run the Electron App
After setting up the above code, build and run your Electron app:

```bash
npm start
```
This will open an Electron window displaying the Apache Superset dashboard. Adjust the URLs and authentication logic as needed to fit your specific Superset setup.
