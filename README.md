# URL Shortener

A high-performance URL shortening service built with Go, featuring a clean architecture, SQLite storage, and comprehensive logging.

## 🚀 Features

- **URL Shortening**: Create short aliases for long URLs
- **Custom Aliases**: Optionally specify custom aliases for URLs
- **Auto-generated Aliases**: Automatically generate 6-character random aliases when not specified
- **URL Validation**: Comprehensive URL validation using go-playground/validator
- **Authentication**: Basic HTTP authentication for URL creation
- **Graceful Shutdown**: Proper server shutdown with timeout handling
- **Structured Logging**: Beautiful colored logging in development, JSON in production
- **SQLite Storage**: Lightweight, file-based database storage
- **Clean Architecture**: Well-structured codebase with separation of concerns
- **Comprehensive Testing**: Unit tests with mocking support
- **Configuration Management**: YAML-based configuration with environment variable support

## 🏗️ Architecture

The project follows clean architecture principles with the following structure:

```
url-shortener/
├── cmd/url-shortener/          # Application entry point
├── config/                     # Configuration files
├── internal/
│   ├── config/                 # Configuration management
│   ├── http-server/            # HTTP server and handlers
│   │   ├── handlers/           # Request handlers
│   │   │   ├── redirect/       # URL redirection handler
│   │   │   └── url/save/       # URL creation handler
│   │   └── middleware/         # HTTP middleware
│   ├── lib/                    # Shared libraries
│   │   ├── api/                # API response utilities
│   │   ├── logger/             # Logging utilities
│   │   └── random/             # Random string generation
│   └── storage/                # Data storage layer
│       └── sqlite/             # SQLite implementation
├── storage/                    # Database files
└── tests/                      # Integration tests
```

## 📋 Prerequisites

- Go 1.23.0 or higher
- SQLite3 (included via go-sqlite3 driver)

## 🛠️ Installation

1. **Clone the repository**:
   ```bash
   git clone <repository-url>
   cd url-shortener
   ```

2. **Install dependencies**:
   ```bash
   go mod download
   ```

3. **Set up configuration**:
   ```bash
   cp config/local.yaml config/local.yaml.example
   # Edit config/local.yaml with your settings
   ```

4. **Run the application**:
   ```bash
   CONFIG_PATH=./config/local.yaml go run cmd/url-shortener/main.go
   ```

## ⚙️ Configuration

The application uses YAML configuration files. Create a `config/local.yaml` file:

```yaml
env: local                    # Environment: local, dev, prod
storage_path: "./storage/storage.db"  # SQLite database path
http_server:
  address: "localhost:8082"   # Server address and port
  timeout: 4s                 # Request timeout
  idle_timeout: 60s           # Idle connection timeout
  user: "admin"               # Basic auth username
  password: "123456"          # Basic auth password
```

### Environment Variables

You can override configuration values using environment variables:

- `CONFIG_PATH`: Path to the configuration file (required)
- `HTTP_SERVER_PASSWORD`: HTTP server password

## 🚀 Usage

### Starting the Server

```bash
CONFIG_PATH=./config/local.yaml go run cmd/url-shortener/main.go
```

The server will start on the configured address (default: `localhost:8082`).

### API Endpoints

#### 1. Create Short URL

**POST** `/url`

Creates a short URL alias for a given URL.

**Authentication**: Basic HTTP authentication required

**Request Body**:
```json
{
  "url": "https://example.com/very/long/url/that/needs/shortening",
  "alias": "my-custom-alias"  // Optional: if not provided, a random 6-character alias will be generated
}
```

**Response**:
```json
{
  "status": "OK",
  "alias": "my-custom-alias"
}
```

**Example with curl**:
```bash
curl -X POST http://localhost:8082/url \
  -H "Content-Type: application/json" \
  -u "admin:123456" \
  -d '{"url": "https://example.com/very/long/url", "alias": "test123"}'
```

#### 2. Redirect to Original URL

**GET** `/{alias}`

Redirects to the original URL associated with the given alias.

**Example**:
```bash
curl -I http://localhost:8082/test123
# Returns: HTTP/1.1 302 Found
# Location: https://example.com/very/long/url
```

### Error Responses

The API returns structured error responses:

```json
{
  "status": "Error",
  "error": "field URL is a required field"
}
```

Common error scenarios:
- **400 Bad Request**: Invalid URL format or missing required fields
- **401 Unauthorized**: Missing or invalid authentication
- **404 Not Found**: Alias not found
- **409 Conflict**: URL already exists with the same alias

## 🧪 Testing

### Running Tests

```bash
# Run all tests
go test ./...

# Run tests with coverage
go test -cover ./...

# Run specific test
go test ./internal/http-server/handlers/url/save
```

### Test Structure

- **Unit Tests**: Located alongside source files (`*_test.go`)
- **Mock Generation**: Uses `mockery` for generating mocks
- **Test Utilities**: Includes test helpers and discard loggers

## 🔧 Development

### Project Structure

- **Handlers**: HTTP request handlers in `internal/http-server/handlers/`
- **Middleware**: HTTP middleware in `internal/http-server/middleware/`
- **Storage**: Database layer in `internal/storage/`
- **Configuration**: Config management in `internal/config/`
- **Utilities**: Shared libraries in `internal/lib/`

### Adding New Features

1. **Create handlers** in `internal/http-server/handlers/`
2. **Add storage methods** in `internal/storage/`
3. **Update routes** in `cmd/url-shortener/main.go`
4. **Add tests** alongside your code
5. **Update configuration** if needed

### Logging

The application uses structured logging with different levels:

- **Local Environment**: Pretty-printed colored logs
- **Development**: JSON format with debug level
- **Production**: JSON format with info level

### Database Schema

```sql
CREATE TABLE url (
    id INTEGER PRIMARY KEY,
    alias TEXT NOT NULL UNIQUE,
    url TEXT NOT NULL
);
CREATE INDEX idx_alias ON url (alias);
```

## 🚀 Deployment

### Production Deployment

1. **Build the binary**:
   ```bash
   go build -o url-shortener cmd/url-shortener/main.go
   ```

2. **Set up production configuration**:
   ```yaml
   env: prod
   storage_path: "/var/lib/url-shortener/storage.db"
   http_server:
     address: ":8080"
     timeout: 30s
     idle_timeout: 120s
     user: "your-username"
     password: "your-secure-password"
   ```

3. **Run with systemd** (example service file):
   ```ini
   [Unit]
   Description=URL Shortener Service
   After=network.target

   [Service]
   Type=simple
   User=url-shortener
   WorkingDirectory=/opt/url-shortener
   Environment=CONFIG_PATH=/opt/url-shortener/config/prod.yaml
   ExecStart=/opt/url-shortener/url-shortener
   Restart=always

   [Install]
   WantedBy=multi-user.target
   ```

### Docker Deployment

```dockerfile
FROM golang:1.23-alpine AS builder
WORKDIR /app
COPY . .
RUN go build -o url-shortener cmd/url-shortener/main.go

FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/url-shortener .
COPY --from=builder /app/config ./config
EXPOSE 8080
CMD ["./url-shortener"]
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🐛 Known Issues

- TODO: Add DELETE endpoint for URL removal
- TODO: Add URL expiration functionality
- TODO: Add rate limiting
- TODO: Add metrics and monitoring

## 📞 Support

For support and questions, please open an issue in the GitHub repository.