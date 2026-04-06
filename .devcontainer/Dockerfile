# Start with a lightweight Debian Linux environment that has Node.js and npm pre-installed
FROM node:22-bookworm-slim

# Install tmux, git, and any other standard utilities you want
RUN apt-get update && apt-get install -y \
    tmux \
    git \
    nano \
    && rm -rf /var/lib/apt/lists/*


# enable mouse scrolling and increase the scrollback history limit
RUN echo -e "set-option -g history-limit 100000\nset -g mouse on" > ~/.tmux.conf
# RUN echo -e "set-option -g history-limit 100000\nset -g mouse on" > ~/.tmux.conf && tmux source-file ~/.tmux.conf

# Install Claude Code using npm
RUN npm install -g @anthropic-ai/claude-code

ENV ANTHROPIC_API_KEY=HNRGXC9ZFDZA4CMKD3H4J2MN
ENV ANTHROPIC_BASE_URL=https://app.dataannotation.tech/api/llm_proxy/org_b

# The Node image conveniently provides a non-root user named 'node'. 
# Switching to this user prevents file permission headaches when sharing files with Windows.
USER node
WORKDIR /workspace
