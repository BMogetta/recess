# Build stage (pinned to the build host's native arch; cross-compile via GOARCH).
FROM --platform=$BUILDPLATFORM golang:1.26-alpine@sha256:80fbb8f9b2fa541a7d34378f1ad10f4f1c433817c4ed39ddb3e2f3ec3e961271 AS builder
ARG TARGETOS
ARG TARGETARCH

WORKDIR /app

# 1. Copy module definitions first to leverage Docker cache
# These paths are relative to the build context (project root)
COPY shared/go.mod shared/go.sum ./shared/
COPY services/game-server/go.mod services/game-server/go.sum ./services/game-server/

# 2. Download dependencies
# GOWORK=off ensures Go doesn't look for the root go.work file
WORKDIR /app/services/game-server
RUN GOWORK=off go mod download

# 3. Copy the actual source code
WORKDIR /app
COPY shared/ ./shared/
COPY services/game-server/ ./services/game-server/

# 4. Build the binary
WORKDIR /app/services/game-server
RUN CGO_ENABLED=0 GOOS=$TARGETOS GOARCH=$TARGETARCH GOWORK=off go build -o /bin/server ./cmd/server

# --- Run stage ---
FROM alpine:3.24@sha256:28bd5fe8b56d1bd048e5babf5b10710ebe0bae67db86916198a6eec434943f8b
RUN apk --no-cache add ca-certificates tzdata wget

COPY --from=builder /bin/server /bin/server

RUN adduser -D -u 1001 appuser
USER appuser

CMD ["/bin/server"]