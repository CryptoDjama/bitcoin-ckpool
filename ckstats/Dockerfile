# Use an Ubuntu base image
FROM ubuntu:22.04

# Let’s define some build arguments
ARG TARGETARCH
ARG DGB_VERSION=8.22.1

ENV FNM_PATH="/root/.local/share/fnm"

# Install necessary dependencies
RUN apt-get update && apt-get install -y \
    git curl build-essential net-tools unzip jq cron nano \
    && rm -rf /var/lib/apt/lists/*

# Cron job for ckstats
COPY ckstatscron /etc/cron.d/ckstatscron
RUN chmod 0644 /etc/cron.d/ckstatscron
RUN crontab /etc/cron.d/ckstatscron
RUN touch /var/log/cron.log

# Install fnm, Node.js, and pnpm
RUN curl -o- https://fnm.vercel.app/install | bash && \
    # Source the fnm environment
    export PATH="$FNM_PATH:$PATH" && \
    eval "`fnm env`" && \
    # Download and install Node.js:
    fnm install 22 && \
    # Verify the Node.js version:
    node -v # Should print "v22.14.0".

# Set the working directory
WORKDIR /app

# Clone the ckstats repository
RUN git clone https://github.com/mrv777/ckstats.git

# Install the dependencies
WORKDIR /app/ckstats
RUN ls -la
    # Source the fnm environment
RUN export PATH="$FNM_PATH:$PATH" && \
    eval "`fnm env`" && \
    # Download and install pnpm:
    corepack prepare pnpm@latest --activate && \
    corepack enable pnpm && \
    pnpm install

# Download and install Bitcoin Core
# We pick x86_64 or arm64 archives depending on $TARGETARCH
RUN if [ "$TARGETARCH" = "amd64" ]; then \
      curl -L -o bitcoin.tar.gz https://bitcoin.org/bin/bitcoin-core-27.0/bitcoin-27.0-x86_64-linux-gnu.tar.gz; \
    elif [ "$TARGETARCH" = "arm64" ]; then \
      curl -L -o bitcoin.tar.gz https://bitcoin.org/bin/bitcoin-core-27.0/bitcoin-27.0-arm-linux-gnueabihf.tar.gz; \
    else \
      echo "Unsupported or unknown architecture: $TARGETARCH"; \
      exit 1; \
    fi \
    && tar -xzvf bitcoin.tar.gz --strip-components=1 -C /usr/local \
    && rm bitcoin.tar.gz

# Expose the port
EXPOSE 4000

# Start the application

COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]