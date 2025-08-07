FROM node:lts-slim

# Configure default locale (important for chrome-headless-shell).
ENV LANG=de_DE.UTF-8 
# UID of the non-root user 'pptruser'
ENV PPTRUSER_UID=10042
# Attempts to start a new DBUS session if none is present
ENV DBUS_SESSION_BUS_ADDRESS=autolaunch:
# Skip Puppeteer Chromium download since we're installing it via apt
ENV PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true
# Set the path to the installed Chromium executable
ENV PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium

# Install Chromium, necessary dependencies and fonts to support major charsets
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
       ca-certificates \
       curl \
       chromium \
       chromium-sandbox \
       fonts-ipafont-gothic \
       fonts-wqy-zenhei \
       fonts-thai-tlwg \
       fonts-khmeros \
       fonts-kacst \
       fonts-freefont-ttf \
       dbus \
       dbus-x11 \
       libnss3 \
       libxss1 \
       libasound2 \
       libxtst6 \
       libgtk-3-0 \
       libgbm1 \
    && update-ca-certificates \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Add non-root user 'pptruser'
RUN groupadd -r pptruser \
    && useradd -u $PPTRUSER_UID -rm -g pptruser -G audio,video pptruser

USER $PPTRUSER_UID
WORKDIR /home/pptruser

# Configure a per-user global npm prefix so we can install -g without root,
# and ensure Node finds the modules via NODE_PATH.
ENV NPM_CONFIG_PREFIX=/home/pptruser/.npm-global
ENV PATH=$NPM_CONFIG_PREFIX/bin:$PATH
ENV NODE_PATH=$NPM_CONFIG_PREFIX/lib/node_modules

# Create prefix dir & install Puppeteer packages globally into it
RUN mkdir -p $NPM_CONFIG_PREFIX/lib/node_modules \
    && npm install -g puppeteer puppeteer-core @puppeteer/browsers --no-save

# Generate THIRD_PARTY_NOTICES using chrome --credits.
RUN chromium --headless --disable-gpu --credits > THIRD_PARTY_NOTICES 2>/dev/null || true