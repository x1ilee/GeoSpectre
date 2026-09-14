# Live Location Tracker

A browser-based location-sharing demo built with Node.js, Express, and Socket.IO. It receives coordinates from a browser and displays the latest position on a map without refreshing the page.

**made by @x1ilee** 


## Features

- Login-protected dashboard with a list of location sessions.
- Live map markers using Leaflet and OpenStreetMap tiles.
- Browser location requests approximately every five seconds while the sharing page runs.
- Socket.IO updates for new sessions and incoming coordinates.
- In-memory storage of the latest location for each browser identifier.

Accuracy depends on the device, browser, permissions, and location services. Continuous background tracking is not guaranteed.

## What the current interface actually does

The participant page is titled **Weather App**, but its weather values are placeholders. Clicking **Check Weather** starts requesting browser location and sending coordinates to the server. It does not fetch a weather forecast.

Before using this demo with others, replace that wording with an explicit location-sharing explanation and add visible start/stop controls. Participants should understand who receives their location and agree to sharing it.

Starting the server also automatically requests a **Cloudflare tunnel**, which can expose the application through a public URL. Startup is not local-only.

## Requirements

- Node.js and npm. The repository does not declare a supported Node.js version range.
- A browser with geolocation support and location permission.
- Internet access for dependencies, tunnel creation, external scripts, and map tiles.

## Setup

### 1. Open the project folder

Open a terminal in the downloaded folder containing `package.json` and `server.js`.

### 2. Install dependencies

```bash
npm install
```

### 3. Review the settings

Configuration lives in `config.js`:

| Setting | Description |
| --- | --- |
| `port` | Local port, defaulting to `6589`. The `PORT` environment variable takes precedence. |
| `username` | Dashboard login username. |
| `password` | Dashboard login password. |
| `token` | Static value used to check the login cookie. |

The supplied login is `admin` / `admin`. Replace the credentials and supplied token before use. This alone does not resolve the access-control limitations below.

### 4. Run the application

```bash
npm start
```

The server starts and requests its tunnel. After tunnel setup succeeds, the terminal prints the `LOCAL` and `REMOTE` addresses.

With default settings, the local dashboard is at `http://localhost:6589`. Sign in using the credentials from `config.js`.

Press **Ctrl+C** in the terminal to stop the server.

## How location updates work

1. The participant page creates a browser identifier in `localStorage`, or reuses an existing one.
2. Clicking its button starts location requests, subject to browser permission.
3. Successful requests send the identifier, latitude, and longitude to the server.
4. The server stores the latest coordinates and broadcasts an update through Socket.IO.
5. The dashboard lists new sessions, and the selected map moves its marker as updates arrive.

An identifier represents browser storage, not a verified person. Clearing browser storage can create a new identifier. Closing the sharing page stops its script; revoking location permission prevents further successful location reads. The current page has no dedicated stop-sharing button.

## Project structure

```text
live-location-tracker/
├── config.js          # Port, credentials, and cookie token
├── package.json       # Dependencies and npm scripts
├── server.js          # Server, templates, sockets, and tunnel startup
├── router.js          # Login, coordinate intake, dashboard, and map routes
└── views/
    ├── home.html      # Dashboard and session list
    ├── login.html     # Login form
    ├── map.html       # Live map
    └── weather.html   # Location collection page with weather wording
```

The server also serves static assets from the `public` directory.

## Current limitations

- **Access control:** dashboard and map HTTP pages check a cookie, but Socket.IO connections are not authenticated. Coordinate updates are broadcast to connected sockets, so page login does not protect the full data flow.
- **Input validation:** the coordinate submission endpoint does not authenticate participants or validate coordinates.
- **Authentication:** credentials are plaintext configuration values, and login uses a static cookie token.
- **Retention:** coordinates are stored in memory and printed to the console. A server restart clears the session list, but terminal or hosting logs may retain coordinates.
- **Session status:** entries do not expire when sharing stops. A listed session does not prove that sharing is still active.
- **History:** no database, route history, or playback feature is implemented.
- **Participant controls:** clear disclosure, a stop-sharing control, and improved error handling are needed before use with others.

## Troubleshooting

| Problem | What to check |
| --- | --- |
| `npm` is not recognized | Confirm Node.js and npm are installed and available in PATH. Reopen the terminal after installation. |
| Port is already in use | Stop the process using it, or change the configured port. |
| No remote address appears | Check terminal errors and internet access. Address logging happens after tunnel setup succeeds. |
| Login fails | Check `config.js` and restart the server after configuration changes. |
| No location appears | Check site permission, device location services, and browser errors. Geolocation generally requires HTTPS or localhost. |
| Map tiles do not load | Check internet access and blocked external script or tile requests. |
| Old sessions remain visible | Entries remain until server restart; no expiry mechanism is implemented. |

## Development

`npm start` runs the server. `npm run dev` invokes `nodemon`, but that tool is not included in the project dependencies and requires a separate installation. No automated test script is defined in `package.json`.

## Credits and license

- **Interface and documentation customization:** @x1ilee
- **Libraries:** Express, Socket.IO, Tarkine, Leaflet, and the `cloudflared` package.
- **Map tiles:** OpenStreetMap.

`package.json` declares the **ISC** license. Preserve applicable upstream license notices when redistributing the project.
