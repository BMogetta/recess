# Build stage (pinned to the build host's native arch; cross-compile via GOARCH).
FROM --platform=$BUILDPLATFORM golang:1.27-alpine@sha256:4c9fe60190a2a3350ddc51de80d0224b8a6698d12bdfc999fee45ea9d6c46dbc AS builder
ARG TARGETOS
ARG TARGETARCH
WORKDIR /app

# 1. Copy module definitions first to leverage Docker cache
# Includes shared and any other local module the rating-service depends on.
COPY shared/go.mod shared/go.sum ./shared/
COPY services/rating-service/go.mod services/rating-service/go.sum ./services/rating-service/

# 2. Download dependencies
# We bypass the root go.work to ensure the build is self-contained.
WORKDIR /app/services/rating-service
RUN GOWORK=off go mod download

# 3. Copy the actual source code
WORKDIR /app
COPY shared/ ./shared/
COPY services/rating-service/ ./services/rating-service/

# 4. Build the binary
# Targeted to ./cmd/server as requested.
WORKDIR /app/services/rating-service
RUN CGO_ENABLED=0 GOOS=$TARGETOS GOARCH=$TARGETARCH GOWORK=off go build -o /bin/rating-service ./cmd/server

# --- Run stage ---
FROM alpine:3.24@sha256:28bd5fe8b56d1bd048e5babf5b10710ebe0bae67db86916198a6eec434943f8b
RUN apk --no-cache add ca-certificates tzdata wget

WORKDIR /app

# Copy the compiled binary from the builder stage
COPY --from=builder /bin/rating-service /app/rating-service

RUN adduser -D -u 1001 appuser
USER appuser

EXPOSE 8085 9085

ENTRYPOINT ["/app/rating-service"]