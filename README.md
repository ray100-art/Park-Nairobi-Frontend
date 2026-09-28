# ParkNairobi — Smart Parking Frontend

The web frontend for **ParkNairobi**, a smart-parking platform for Kenyan towns. Drivers find
a free bay on a live map, reserve it, pay with **M-Pesa**, and manage their bookings. Admins
watch occupancy and manage users. Bays are tracked through entry and exit sensor events.

**Live demo:** https://ray100-art.github.io/Park-Nairobi-Frontend/ (the map and pages load
without a backend; sign-in, bookings and payments need the API running).

The API behind it is a Java 21 / Spring Boot 3 service with JWT auth, role-based access,
Flyway migrations, STOMP WebSocket updates, and an M-Pesa callback that is authenticated and
idempotent (a payment is only settled once): see
[Park-Nairobi-Backend](https://github.com/ray100-art/Park-Nairobi-Backend).

The demo data covers parking areas in Nairobi (CBD, Westlands, Upper Hill, Karen,
Thika Road), Mombasa, Kisumu, Nakuru, Eldoret, Meru, Embu and Chuka.

## Screens

| Page | For | What it does |
|---|---|---|
| `index.html` | Everyone | Sign in and register, then route by role |
| `dashboard.html` | Drivers | Leaflet/OpenStreetMap map of bays, nearest bays by geolocation, and reservations |
| `bookings.html` | Drivers | Bookings, cancellation, and M-Pesa payment with live status polling |
| `Profile.html` | Drivers | Account details |
| `admin.html` | Admins | Occupancy summary, all bookings, users, freeing slots |

## How it works

- **One API client** in `js/api.js` wraps every call: auth, slots, bookings, M-Pesa and
  sensors. It attaches the JWT and escapes user-supplied text before rendering it.
- **Environment-aware API base.** `js/config.js` targets `localhost:8080` in development and
  the same origin in production. On Netlify, `build-config.js` writes the address from the
  `API_BASE` environment variable.
- **Payments.** `POST /api/mpesa/pay` starts an STK Push, then the page polls
  `/api/mpesa/status/{checkoutId}` until the payment settles.
- **Sensors.** `/api/sensor/entry` and `/api/sensor/exit` record cars arriving and leaving,
  which keeps bay status accurate.

## Tech stack

| Area | Tools |
|---|---|
| UI | HTML5, CSS3, vanilla JavaScript (ES6+) |
| Maps | Leaflet, OpenStreetMap, browser Geolocation API |
| Deployment | Docker (`nginx:1.27-alpine`), Nginx reverse proxy for `/api` and WebSocket `/ws`, Netlify |

## Running locally

With the backend API on port 8080, serve this folder statically:

```bash
python -m http.server 5500
# open http://localhost:5500
```

### With Docker

```bash
docker build -t parknairobi-web .
docker run -p 80:80 parknairobi-web
```

The Nginx config proxies `/api/` and `/ws` to a container named `api` on port 8080. Run both
containers on the same Docker network, or change `proxy_pass` in `nginx.conf`.

### On Netlify

Set `API_BASE` in *Site settings → Environment variables*. The build step writes it into
`js/config.js`, and `netlify.toml` maps `/admin`, `/dashboard`, `/bookings` and `/profile` to
their pages.

## License

[MIT](LICENSE)
