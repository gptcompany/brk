# syntax=docker/dockerfile:1.7
FROM rust:1.94-bookworm AS builder

RUN apt-get update && apt-get install -y pkg-config libssl-dev && rm -rf /var/lib/apt/lists/*

WORKDIR /app/brk

COPY Cargo.toml Cargo.lock rust-toolchain.toml ./
COPY .cargo .cargo
COPY crates crates
COPY website website
COPY modules modules
COPY --from=anydb / /app/anydb

RUN cargo build --release --locked -p brk_cli

FROM debian:bookworm-slim

RUN apt-get update && apt-get install -y ca-certificates curl && rm -rf /var/lib/apt/lists/*

RUN useradd -m -s /bin/bash brk

COPY --from=builder /app/brk/target/release/brk /usr/local/bin/brk

USER brk

RUN mkdir -p /home/brk/.brk

EXPOSE 3110

WORKDIR /home/brk

ENTRYPOINT ["brk"]
