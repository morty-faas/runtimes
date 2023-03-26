FROM --platform=${TARGETPLATFORM:-linux/amd64} ghcr.io/polyxia-org/alpha:v0.2.4 as get-alpha
FROM --platform=${TARGETPLATFORM:-linux/amd64} python:3-alpine


ARG TARGETPLATFORM
ARG ADDITIONAL_PACKAGE

RUN apk --no-cache add curl ca-certificates ${ADDITIONAL_PACKAGE}

# Get Alpha and make it executable in the final build image
COPY --from=get-alpha /alpha /usr/bin/alpha
RUN chmod +x /usr/bin/alpha


# Add user function and install dependencies
RUN mkdir -p /app/function

WORKDIR /app
COPY  index.py           .
COPY  requirements.txt   .
RUN pip install --no-cache-dir -r requirements.txt

# Install function dependencies in a separate layer to allow caching
RUN touch ./function/__init__.py
WORKDIR /app/function/
COPY  function/requirements.txt	.
RUN pip install --no-cache-dir -r requirements.txt

# Copy the function
COPY  function/   .

# Mandatory, will be used by Alpha
RUN echo 'export ALPHA_PROCESS_COMMAND="/usr/local/bin/python /app/index.py"' >> /app/env.sh

WORKDIR /
