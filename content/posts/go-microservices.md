---
title: "Costruire Microservizi in Go: Best Practices e Pattern"
date: 2025-01-26T14:00:00+01:00
author: "Andrea Morosati"
tags: ["go", "microservices", "architecture", "backend", "best-practices"]
draft: false
---

# Costruire Microservizi in Go: Best Practices e Pattern

Go è diventato uno dei linguaggi preferiti per lo sviluppo di microservizi grazie alla sua semplicità, performance e supporto nativo per la concorrenza. In questo articolo condivido le mie esperienze e best practices per costruire microservizi robusti e scalabili.

## Perché Go per i Microservizi?

### Vantaggi Principali

- **Performance**: Compilazione nativa e gestione efficiente della memoria
- **Concorrenza**: Goroutines e channels per gestione asincrona
- **Semplicità**: Sintassi pulita e standard library completa
- **Deployment**: Binari singoli, facile containerizzazione

## Architettura di Base

### Struttura del Progetto

```
microservice/
├── cmd/
│   └── server/
│       └── main.go
├── internal/
│   ├── handler/
│   ├── service/
│   ├── repository/
│   └── middleware/
├── pkg/
│   ├── database/
│   ├── config/
│   └── logger/
├── api/
│   └── proto/
├── docker/
└── go.mod
```

### Pattern Architetturali

#### 1. Clean Architecture

```go
// internal/handler/user.go
type UserHandler struct {
    userService service.UserService
}

func (h *UserHandler) CreateUser(c *gin.Context) {
    var user domain.User
    if err := c.ShouldBindJSON(&user); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }
    
    result, err := h.userService.Create(c.Request.Context(), user)
    if err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
        return
    }
    
    c.JSON(http.StatusCreated, result)
}
```

#### 2. Dependency Injection

```go
// cmd/server/main.go
func main() {
    // Configurazione
    config := config.Load()
    
    // Database
    db := database.NewConnection(config.Database)
    
    // Repository
    userRepo := repository.NewUserRepository(db)
    
    // Service
    userService := service.NewUserService(userRepo)
    
    // Handler
    userHandler := handler.NewUserHandler(userService)
    
    // Router
    router := gin.Default()
    router.POST("/users", userHandler.CreateUser)
    
    router.Run(":8080")
}
```

## Best Practices

### 1. Gestione Configurazione

```go
// pkg/config/config.go
type Config struct {
    Server   ServerConfig   `mapstructure:"server"`
    Database DatabaseConfig `mapstructure:"database"`
    Redis    RedisConfig    `mapstructure:"redis"`
}

type ServerConfig struct {
    Port    string `mapstructure:"port"`
    Timeout int    `mapstructure:"timeout"`
}

func Load() *Config {
    viper.SetConfigName("config")
    viper.SetConfigType("yaml")
    viper.AddConfigPath(".")
    
    if err := viper.ReadInConfig(); err != nil {
        log.Fatal("Error reading config file", err)
    }
    
    var config Config
    if err := viper.Unmarshal(&config); err != nil {
        log.Fatal("Error unmarshaling config", err)
    }
    
    return &config
}
```

### 2. Logging Strutturato

```go
// pkg/logger/logger.go
import "go.uber.org/zap"

func NewLogger() *zap.Logger {
    config := zap.NewProductionConfig()
    config.OutputPaths = []string{"stdout", "logs/app.log"}
    
    logger, err := config.Build()
    if err != nil {
        log.Fatal("Error creating logger", err)
    }
    
    return logger
}

// Utilizzo
logger.Info("User created successfully",
    zap.String("user_id", user.ID),
    zap.String("email", user.Email),
)
```

### 3. Middleware per Observability

```go
// internal/middleware/logging.go
func LoggingMiddleware(logger *zap.Logger) gin.HandlerFunc {
    return func(c *gin.Context) {
        start := time.Now()
        
        c.Next()
        
        duration := time.Since(start)
        logger.Info("Request processed",
            zap.String("method", c.Request.Method),
            zap.String("path", c.Request.URL.Path),
            zap.Int("status", c.Writer.Status()),
            zap.Duration("duration", duration),
        )
    }
}
```

## Comunicazione tra Servizi

### 1. REST API

```go
// internal/client/user_client.go
type UserClient struct {
    baseURL string
    client  *http.Client
}

func (c *UserClient) GetUser(id string) (*User, error) {
    resp, err := c.client.Get(fmt.Sprintf("%s/users/%s", c.baseURL, id))
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    
    var user User
    if err := json.NewDecoder(resp.Body).Decode(&user); err != nil {
        return nil, err
    }
    
    return &user, nil
}
```

### 2. gRPC

```protobuf
// api/proto/user.proto
syntax = "proto3";

package user;

service UserService {
    rpc GetUser(GetUserRequest) returns (User);
    rpc CreateUser(CreateUserRequest) returns (User);
}

message User {
    string id = 1;
    string email = 2;
    string name = 3;
}
```

## Testing

### 1. Unit Tests

```go
// internal/service/user_service_test.go
func TestUserService_Create(t *testing.T) {
    // Setup
    mockRepo := &mocks.UserRepository{}
    service := NewUserService(mockRepo)
    
    user := domain.User{
        Email: "test@example.com",
        Name:  "Test User",
    }
    
    mockRepo.On("Create", mock.Anything, user).Return(&user, nil)
    
    // Execute
    result, err := service.Create(context.Background(), user)
    
    // Assert
    assert.NoError(t, err)
    assert.Equal(t, user.Email, result.Email)
    mockRepo.AssertExpectations(t)
}
```

### 2. Integration Tests

```go
// tests/integration/user_test.go
func TestUserAPI_CreateUser(t *testing.T) {
    // Setup test server
    router := setupTestRouter()
    server := httptest.NewServer(router)
    defer server.Close()
    
    // Test data
    userData := map[string]interface{}{
        "email": "test@example.com",
        "name":  "Test User",
    }
    
    jsonData, _ := json.Marshal(userData)
    
    // Execute request
    resp, err := http.Post(server.URL+"/users", "application/json", bytes.NewBuffer(jsonData))
    assert.NoError(t, err)
    assert.Equal(t, http.StatusCreated, resp.StatusCode)
}
```

## Deployment e Containerizzazione

### Dockerfile

```dockerfile
# Dockerfile
FROM golang:1.21-alpine AS builder

WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main ./cmd/server

FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/

COPY --from=builder /app/main .
COPY --from=builder /app/config ./config

EXPOSE 8080
CMD ["./main"]
```

### Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  user-service:
    build: .
    ports:
      - "8080:8080"
    environment:
      - DB_HOST=postgres
      - REDIS_HOST=redis
    depends_on:
      - postgres
      - redis
  
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: users
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
  
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  postgres_data:
```

## Monitoring e Observability

### 1. Health Checks

```go
// internal/handler/health.go
func HealthCheck(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{
        "status": "healthy",
        "timestamp": time.Now().Unix(),
        "version": "1.0.0",
    })
}
```

### 2. Metrics con Prometheus

```go
// pkg/metrics/metrics.go
import "github.com/prometheus/client_golang/prometheus"

var (
    httpRequestsTotal = prometheus.NewCounterVec(
        prometheus.CounterOpts{
            Name: "http_requests_total",
            Help: "Total number of HTTP requests",
        },
        []string{"method", "endpoint", "status"},
    )
    
    httpRequestDuration = prometheus.NewHistogramVec(
        prometheus.HistogramOpts{
            Name: "http_request_duration_seconds",
            Help: "HTTP request duration in seconds",
        },
        []string{"method", "endpoint"},
    )
)
```

## Conclusione

Costruire microservizi in Go richiede attenzione all'architettura, testing e observability. Le best practices presentate in questo articolo forniscono una base solida per sviluppare servizi robusti e scalabili.

### Punti Chiave

- ✅ Utilizza Clean Architecture per separazione delle responsabilità
- ✅ Implementa logging strutturato e monitoring
- ✅ Scrivi test unitari e di integrazione
- ✅ Containerizza con Docker per deployment consistente
- ✅ Monitora performance e health dei servizi

### Risorse Utili

- [Go by Example](https://gobyexample.com/)
- [Effective Go](https://golang.org/doc/effective_go.html)
- [Go Web Examples](https://gowebexamples.com/)

---

**Tag**: #go #microservices #architecture #backend #best-practices #docker #testing 