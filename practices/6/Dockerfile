FROM golang:1.24-alpine AS build

WORKDIR /app
COPY main.go .
RUN CGO_ENABLED=0 go build -o app main.go

FROM alpine:3.21
WORKDIR /app
COPY --from=build /app/app ./app
CMD ["./app"]
