FROM --platform=${TARGETPLATFORM:-linux/amd64} ghcr.io/morty-faas/alpha:v1.0.1 as get-alpha
FROM --platform=${TARGETPLATFORM:-linux/amd64} rust:1.67-alpine as build

ARG TARGETPLATFORM

RUN apk add --no-cache build-base

RUN mkdir -p /handler/function

# Install the function dependencies
# to allow caching
RUN cargo init --lib handler/function
WORKDIR /handler/function

RUN mv ./src/lib.rs ./handler.rs
COPY ./function/Cargo.* ./
RUN cargo build --release
RUN rm *.rs

# Compile the function code
COPY ./function/ ./
RUN rm ./target/release/deps/function*
RUN cargo build --release

# Install entrypoint dependencies
# to allow caching
WORKDIR /handler
RUN cargo init --bin . && mv ./src/main.rs ./main.rs

COPY ./Cargo.* ./
RUN cargo build --release
RUN rm *.rs

# Compile the entrypoint
COPY . .
RUN rm ./target/release/deps/handler*
RUN cargo build --release

FROM alpine:3.17.2

ARG ADDITIONAL_PACKAGE
RUN apk --no-cache add curl ca-certificates ${ADDITIONAL_PACKAGE} 

WORKDIR /app

# Get Alpha and make it executable in the final build image
COPY --from=get-alpha /alpha /usr/bin/alpha
RUN chmod +x /usr/bin/alpha

COPY --from=build /handler/target/release/handler .

# Mandatory, will be used by Alpha
RUN echo "export ALPHA_PROCESS_COMMAND=/app/handler" >> /app/env.sh

WORKDIR /
