# Puppeteer Docker Image for ARM64

This repository builds a custom Puppeteer Docker image optimized for ARM64 architecture.

## Features

- **ARM64 native**: Built specifically for ARM64 platforms
- **Chromium included**: Pre-installed Chromium browser 
- **Font support**: Includes fonts for major charsets (Chinese, Japanese, Arabic, Hebrew, Thai)
- **Security focused**: Runs as non-root user `pptruser`
- **Automated builds**: GitHub Actions workflow with scheduled updates

## Quick Start

### Pull the Image

```bash
docker pull ghcr.io/hueske-digital/puppeteer:latest
```

### Run a Puppeteer Script

```bash
docker run -i --init --cap-add=SYS_ADMIN --rm \
  ghcr.io/hueske-digital/puppeteer:latest \
  node -e "
const puppeteer = require('puppeteer');
(async () => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.goto('https://example.com');
  console.log(await page.title());
  await browser.close();
})();
"
```

### Using with Docker Compose

```yaml
services:
  puppeteer:
    image: ghcr.io/hueske-digital/puppeteer:latest
    cap_add:
      - SYS_ADMIN
    init: true
    volumes:
      - ./scripts:/home/pptruser/scripts
    working_dir: /home/pptruser
```

## Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `PUPPETEER_EXECUTABLE_PATH` | Path to Chromium executable | `/usr/bin/chromium` |
| `PUPPETEER_SKIP_CHROMIUM_DOWNLOAD` | Skip Puppeteer's Chromium download | `true` |
| `PPTRUSER_UID` | UID of the non-root user | `10042` |
| `DBUS_SESSION_BUS_ADDRESS` | D-Bus session configuration | `autolaunch:` |

## Important Flags

- `--init`: Properly handles zombie processes
- `--cap-add=SYS_ADMIN`: Required for Chrome's sandbox mode
- `--rm`: Automatically removes container after execution

## Building Locally

```bash
# Clone the repository
git clone <repository-url>
cd puppeteer

# Build the image
docker build -t my-puppeteer:latest ./build

# Run the image
docker run -i --init --cap-add=SYS_ADMIN --rm my-puppeteer:latest node -e "console.log('Hello from Puppeteer!')"
```

## Architecture

This image is built specifically for `linux/arm64` architecture. The GitHub Actions workflow automatically builds and publishes the image using ARM64 runners.

## Automated Updates

The image is automatically rebuilt:
- On every push to the `main` branch
- Weekly on Mondays at midnight UTC via cron schedule
- Manually via workflow dispatch

## Security

- Runs as non-root user `pptruser` (UID: 10042)
- Minimal attack surface with Debian slim base image
- Regular automated security updates through weekly rebuilds

## Troubleshooting

### Permission Issues
If you encounter permission issues, ensure you're using the `--cap-add=SYS_ADMIN` flag.

### Font Issues
The image includes comprehensive font support, but if you need additional fonts, extend the Dockerfile:

```dockerfile
FROM ghcr.io/hueske-digital/puppeteer:latest
USER root
RUN apt-get update && apt-get install -y fonts-your-font && rm -rf /var/lib/apt/lists/*
USER $PPTRUSER_UID
```

## References

- [Puppeteer Docker Guide](https://pptr.dev/guides/docker)
- [Puppeteer Documentation](https://pptr.dev/)
- [GitHub Container Registry](https://ghcr.io/hueske-digital/puppeteer)