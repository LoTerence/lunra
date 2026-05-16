FROM node:22-bookworm-slim

# Set the default shell to bash with pipefail
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Install system utilities (Added curl, unzip, and ca-certificates)
RUN apt-get update && apt-get install -y \
    tmux \
    git \
    nano \
    curl \
    unzip \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*


# Install 'uv' globally (Astral's official install script)
# We move the binaries to /usr/local/bin so all users can access them
RUN curl -LsSf https://astral.sh/uv/install.sh | sh && \
    mv /root/.local/bin/uv /usr/local/bin/ && \
    mv /root/.local/bin/uvx /usr/local/bin/


# Switch to your non-root user
USER node
WORKDIR /home/node

# Add the node user's local bin to the PATH permanently
ENV PATH="/home/node/.local/bin:${PATH}"

# Install Claude Code as the node user
RUN curl -fsSL https://claude.ai/install.sh | bash -s 2.1.143


# ==========================================
# TRACE EXTRACTOR SETUP
# ==========================================

# Download and extract the zip file
# The download link will need to be updated on each Boxing task
# Current version: cli-trace-extractor v1.8.0
RUN curl -o trace_extractor.zip "https://projects-uploaded-files.s3.us-east-2.amazonaws.com/production/item_response_files/_ef924544-553f-48cf-a580-082777d34242_61762969-d51c-479b-9a30-a8d1236d9daf.zip" && \
    unzip trace_extractor.zip -d /home/node/ && \
    rm trace_extractor.zip

# Navigate into the extracted directory and install dependencies
# (Note: uv will automatically download the required version of Python for you)
WORKDIR /home/node/cli-trace-extractor
RUN uv sync

# Generate certificates non-interactively
# We run mitmdump for 2 seconds to generate the certs in ~/.mitmproxy/, then force it to exit.
# The '|| true' ensures the intentional timeout doesn't crash the Docker build.
# (Note: We do this because `uv run mitmproxy` does not work from a Dockerfile)
RUN timeout 10s uv run mitmdump || true

# Switch back to root to install the newly generated certificate to the system
USER root
RUN cp /home/node/.mitmproxy/mitmproxy-ca-cert.pem /usr/local/share/ca-certificates/mitmproxy.crt && \
    update-ca-certificates

# ==========================================
# FINAL WORKSPACE SETUP
# ==========================================

# Switch back to the node user for your day-to-day coding
USER node
WORKDIR /workspace


# Configure tmux settings
# enable mouse scrolling and increase the scrollback history limit
RUN echo "set-option -g history-limit 100000" > /home/node/.tmux.conf && \
    echo "set -g mouse on" >> /home/node/.tmux.conf