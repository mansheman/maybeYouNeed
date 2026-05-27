2026-02-05 09:06

Status:

Tags: [[eWPTX]] [[Server-Side Attacks]]
###### Prerequisites: [[Web Application Architecture Models]]
# Layered Architecture Example (E-commerce)

Example layered model:

- Presentation layer: browser/front-end
- Web server layer: Nginx
- App server layer: uWSGI (business logic)
- API layer: product info API, order management API
- Database layer: MongoDB (separate server)

---

## Web server (Nginx)

- handles HTTP/HTTPS
- serves static assets
- reverse proxy to app server
- may load balance

## App server (uWSGI)

- executes business logic
- handles auth, cart, order workflows
- talks to DB and APIs

## Database (MongoDB)

- product catalog
- customer info
- order history
