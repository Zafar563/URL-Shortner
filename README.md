# URL Shortener

This project is a lightweight, high-performance URL shortening service consisting of a Go-based REST API backend and a clean, responsive web user interface.

---

## 🛠 Tech Stack

The project is built using modern, efficient, and clean tools:

### Backend:
*   **Go (Golang 1.25.0)**: The core language chosen for its speed, low footprint, and concurrency capabilities.
*   **Chi Router (`github.com/go-chi/chi/v5`)**: A lightweight, fast, and idiomatic router for building HTTP services in Go.
*   **Chi Render (`github.com/go-chi/render`)**: Used to easily manage and render JSON request/response payloads.
*   **Cleanenv (`github.com/ilyakaznacheev/cleanenv`)**: A configuration library that parses YAML configurations and environment variables effortlessly.
*   **Godotenv (`github.com/joho/godotenv`)**: Used to load environment variables from a `.env` file during development.
*   **PostgreSQL**: The relational database used to store original URLs and their shortened aliases.
*   **pq (`github.com/lib/pq`)**: A pure Go driver for PostgreSQL.
*   **slog (Go Standard Library)**: Structured logging library configured for different environments (local, dev, prod).

### Frontend:
*   **HTML5 & CSS3**: Visually clean, user-friendly interface styled with the "Outfit" Google Font.
*   **Vanilla JavaScript**: Front-end logic to handle form submissions, call the POST `/url` API endpoint, copy short links to the clipboard, and handle errors.

### Tooling:
*   **golang-migrate**: Database schema migrations managed via Docker.
*   **Makefile**: Shortcuts to easily run database migrations.

---

## 📂 Project Structure

```
URL-shortner/
├── cmd/
│   └── main.go                    # Application entry point
├── config/
│   └── local.yaml                 # Configuration settings for local environment
├── internal/
│   ├── config/
│   │   └── config.go              # Configuration parser and loader
│   ├── http-server/
│   │   └── handlers/
│   │       └── url/
│   │           ├── redirect/      # Handler for redirection (GET /{alias})
│   │           └── save/          # Handler for saving URLs (POST /url)
│   ├── lib/
│   │   ├── api/                   # Common API response structures
│   │   ├── logger/                # Logger helper utilities
│   │   └── random/                # 6-character random alias generator
│   └── storage/
│       ├── postgres/              # PostgreSQL storage logic
│       └── storage.go             # Storage interface and error definitions
├── schema/
│   └── migrations/                # SQL migration files
├── web/
│   ├── index.html                 # Frontend user interface
│   ├── style.css                  # UI styles
│   └── script.js                  # Frontend API client logic
├── .env                           # Environment variables configuration file
├── Makefile                       # Makefile for migrations and utilities
├── go.mod / go.sum                # Go modules and dependencies file
└── README.md                      # Project documentation (this file)
```

---

## ⚙️ Setup and Installation

### 1. Environment Variables (`.env`)
Create or edit a `.env` file in the project root directory:

```env
CONFIG_PATH=config/local.yaml
STORAGE_PATH="user=postgres password=your_password dbname=url_db host=localhost port=5436 sslmode=disable"
```

### 2. Database and Migrations
Ensure Docker is installed and running, then use the following commands to apply migrations:

```bash
# Run database migrations (up)
make migrate-up

# Rollback migrations (down)
make migrate-down
```

### 3. Start the Server
Run the Go application:

```bash
go run cmd/main.go
```

By default, the server will start on `http://localhost:8082`.

---

## 🌐 API Endpoints

### 1. Shorten URL (`POST /url`)
Creates a shortened alias for a long URL.

*   **URL:** `/url`
*   **Method:** `POST`
*   **Headers:** `Content-Type: application/json`
*   **Request Body:**
    ```json
    {
      "url": "https://google.com",
      "alias": "gugl" 
    }
    ```
    *(Note: `alias` is optional. If not provided, a random 6-character alias is generated.)*

*   **Response (Success):**
    ```json
    {
      "status": "OK",
      "alias": "gugl"
    }
    ```

### 2. Redirect to URL (`GET /{alias}`)
Redirects to the original URL associated with the alias.

*   **URL:** `/{alias}` (e.g., `/gugl`)
*   **Method:** `GET`
*   **Response:** Automatically redirects the client to the original URL (Status: `302 Found`).

---

## 💡 Frontend Serving Recommendation

Currently, the backend code (`cmd/main.go`) does not serve static files. To serve the user interface directly through the Go application, you can add a static file server route to your router in `cmd/main.go`:

```go
// Add this route group/handler inside cmd/main.go:
fs := http.FileServer(http.Dir("./web"))
router.Handle("/static/*", http.StripPrefix("/static/", fs))
router.Get("/", func(w http.ResponseWriter, r *http.Request) {
    http.ServeFile(w, r, "./web/index.html")
})
```
