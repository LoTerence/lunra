FROM node:22-bookworm-slim

# 1. Install system utilities (Added curl, unzip, and ca-certificates)
RUN apt-get update && apt-get install -y \
    tmux \
    git \
    nano \
    curl \
    unzip \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# 2. Install Claude Code globally
RUN npm install -g @anthropic-ai/claude-code

# 3. Install 'uv' globally (Astral's official install script)
# We move the binaries to /usr/local/bin so all users can access them
RUN curl -LsSf https://astral.sh/uv/install.sh | sh && \
    mv /root/.local/bin/uv /usr/local/bin/ && \
    mv /root/.local/bin/uvx /usr/local/bin/

# ==========================================
# TRACE EXTRACTOR SETUP
# ==========================================

# 4. Switch to your non-root user to download and set up the project
USER node

# Automatically configure Claude Code settings
RUN mkdir -p /home/node/.claude && \
    echo '{\n  "showThinkingSummaries": true\n}' > /home/node/.claude/settings.json

WORKDIR /home/node

# Download and extract the zip file
RUN curl -o trace_extractor.zip "https://projects-uploaded-files.s3.us-east-2.amazonaws.com/production/item_response_files/_ef924544-553f-48cf-a580-082777d34242_26c4a446-e585-43e0-86c9-830f649a30de.zip" && \
    unzip trace_extractor.zip -d /home/node/ && \
    rm trace_extractor.zip

# Navigate into the extracted directory and install dependencies
# (Note: uv will automatically download the required version of Python for you)
WORKDIR /home/node/cli-trace-extractor
RUN uv sync

# Generate certificates non-interactively
# We run mitmdump for 2 seconds to generate the certs in ~/.mitmproxy/, then force it to exit.
# The '|| true' ensures the intentional timeout doesn't crash the Docker build.
RUN timeout 10s uv run mitmdump || true

# 5. Switch back to root to install the newly generated certificate to the system
USER root
RUN cp /home/node/.mitmproxy/mitmproxy-ca-cert.pem /usr/local/share/ca-certificates/mitmproxy.crt && \
    update-ca-certificates

# ==========================================
# FINAL WORKSPACE SETUP
# ==========================================

# Set Claude Code env vars
ENV ANTHROPIC_API_KEY=HNRGXC9ZFDZA4CMKD3H4J2MN
ENV ANTHROPIC_BASE_URL=https://app.dataannotation.tech/api/llm_proxy/org_b

# 6. Switch back to the node user for your day-to-day coding
USER node
WORKDIR /workspace