# Build stage (pinned to the build host's native arch; cross-compile via GOARCH).
FROM --platform=$BUILDPLATFORM golang:1.26-alpine@sha256:80fbb8f9b2fa541a7d34378f1ad10f4f1c433817c4ed39ddb3e2f3ec3e961271 AS builder
ARG TARGETOS
ARG TARGETARCH
WORKDIR /app

COPY shared/go.mod shared/go.sum ./shared/
COPY services/notification-service/go.mod services/notification-service/go.sum ./services/notification-service/

WORKDIR /app/services/notification-service
RUN GOWORK=off go mod download

WORKDIR /app
COPY shared/ ./shared/
COPY services/notification-service/ ./services/notification-service/

WORKDIR /app/services/notification-service
RUN CGO_ENABLED=0 GOOS=$TARGETOS GOARCH=$TARGETARCH GOWORK=off go build -o /bin/notification-service ./cmd/server

# Run stage
FROM alpine:3.24@sha256:28bd5fe8b56d1bd048e5babf5b10710ebe0bae67db86916198a6eec434943f8b
RUN apk --no-cache add ca-certificates tzdata wget

WORKDIR /app
COPY --from=builder /bin/notification-service /app/notification-service

RUN adduser -D -u 1001 appuser
USER appuser

EXPOSE 8086

ENTRYPOINT ["/app/notification-service"]
