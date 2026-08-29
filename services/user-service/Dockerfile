# Build stage (pinned to the build host's native arch; cross-compile via GOARCH).
FROM --platform=$BUILDPLATFORM golang:1.27-alpine@sha256:4c9fe60190a2a3350ddc51de80d0224b8a6698d12bdfc999fee45ea9d6c46dbc AS builder
ARG TARGETOS
ARG TARGETARCH
WORKDIR /app

# 1. Copy module definitions first to leverage Docker cache
# This ensures "go mod download" only runs when dependencies change.
COPY shared/go.mod shared/go.sum ./shared/
COPY services/user-service/go.mod services/user-service/go.sum ./services/user-service/

# 2. Download dependencies
# GOWORK=off is used here to build the service in isolation using its own mod file
WORKDIR /app/services/user-service
RUN GOWORK=off go mod download

# 3. Copy the actual source code
# We copy shared because user-service depends on it via the go.mod replace directive
WORKDIR /app
COPY shared/ ./shared/
COPY services/user-service/ ./services/user-service/

# 4. Build the binary
# CGO_ENABLED=0 ensures a statically linked binary for the slim alpine image
WORKDIR /app/services/user-service
RUN CGO_ENABLED=0 GOOS=$TARGETOS GOARCH=$TARGETARCH GOWORK=off go build -o /bin/user-service ./cmd/server

# --- Run stage ---
FROM alpine:3.24@sha256:28bd5fe8b56d1bd048e5babf5b10710ebe0bae67db86916198a6eec434943f8b
RUN apk --no-cache add ca-certificates tzdata wget

# Copy the compiled binary from the builder stage
COPY --from=builder /bin/user-service /bin/user-service

RUN adduser -D -u 1001 appuser
USER appuser

ENTRYPOINT ["/bin/user-service"]