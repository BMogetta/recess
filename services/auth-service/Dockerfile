# Build stage (pinned to the build host's native arch; cross-compile via GOARCH).
FROM --platform=$BUILDPLATFORM golang:1.27-alpine@sha256:4c9fe60190a2a3350ddc51de80d0224b8a6698d12bdfc999fee45ea9d6c46dbc AS builder
ARG TARGETOS
ARG TARGETARCH
WORKDIR /app

# 1. Copy module definitions first to leverage Docker cache
# These paths are relative to the build context (project root)
COPY shared/go.mod shared/go.sum ./shared/
COPY services/auth-service/go.mod services/auth-service/go.sum ./services/auth-service/

# 2. Download dependencies
# GOWORK=off ensures Go doesn't look for the root go.work file
WORKDIR /app/services/auth-service
RUN GOWORK=off go mod download

# 3. Copy the actual source code
WORKDIR /app
COPY shared/ ./shared/
COPY services/auth-service/ ./services/auth-service/

# 4. Build the binary
WORKDIR /app/services/auth-service
RUN CGO_ENABLED=0 GOOS=$TARGETOS GOARCH=$TARGETARCH GOWORK=off go build -o /bin/auth-service ./cmd/server

# --- Run stage ---
FROM alpine:3.24@sha256:28bd5fe8b56d1bd048e5babf5b10710ebe0bae67db86916198a6eec434943f8b
RUN apk --no-cache add ca-certificates tzdata wget

COPY --from=builder /bin/auth-service /bin/auth-service

RUN adduser -D -u 1001 appuser
USER appuser

ENTRYPOINT ["/bin/auth-service"]