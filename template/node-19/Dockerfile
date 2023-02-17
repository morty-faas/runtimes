FROM --platform=${TARGETPLATFORM:-linux/amd64} ghcr.io/polyxia-org/alpha:main as get-alpha
FROM --platform=${TARGETPLATFORM:-linux/amd64} node:19-alpine

ARG TARGETPLATFORM
ARG ADDITIONAL_PACKAGE

RUN apk --no-cache add curl ca-certificates openrc util-linux ${ADDITIONAL_PACKAGE} && \
    addgroup -S runner && adduser -S -g runner runner

# Get Alpha and make it executable in the final build image
COPY --from=get-alpha /openrc.sh /etc/init.d/alpha
COPY --from=get-alpha /alpha /usr/bin/alpha
RUN chmod +x /usr/bin/alpha && chmod +x /etc/init.d/alpha

# Temporary. Just for debugging purposes. Without the following lines, we 
# can't enter the microVM through a terminal.
RUN ln -s agetty /etc/init.d/agetty.ttyS0
RUN echo ttyS0 >/etc/securetty
RUN rc-update add agetty.ttyS0 default
RUN echo "root:root" | chpasswd

# Make alpha start on boot
RUN rc-update add alpha default

USER runner

RUN mkdir -p /home/runner/function

# Install entrypoint dependencies
WORKDIR /home/runner
COPY --chown=runner:runner package.json .
RUN npm install

# Copy the function entrypoint
COPY --chown=runner:runner index.js .

# Install function dependencies in a separate layer to allow caching
WORKDIR /home/runner/function
COPY --chown=runner:runner function/*.json .
RUN npm install

# Copy the function
COPY --chown=runner:runner function/ .

WORKDIR /home/runner