# claude-jail

<img src="./prison-claude.png" alt="Screenshot" width="200">

Since Anthropic wont update their docker container to support the native install I will.


### What is this for

- Run claude code in a locked down safe environment
- Use `--allow-dangerously-skip-permissions` without fear
- Includes a firewall sh file for further safety (optional)

### Docker Compose

Build from `docker/` and customize the volume mounts for your setup. Here's a full example `docker-compose.yaml`:

```yaml
services:
  claude:
    image: claude_code:latest
    container_name: claude_code
    build: .
    user: "1000:1000"              # Run as non-root claude user (must match Dockerfile UID/GID)
    restart: unless-stopped
    environment:
      - TERM=xterm-256color        # Terminal color support
      - COLORTERM=truecolor
      - FORCE_COLOR=1
      - CLAUDE_CONFIG_DIR=/home/claude/.claude  # Where Claude looks for config inside the container
    stdin_open: true               # Keep stdin open for interactive use
    tty: true                      # Allocate a pseudo-TTY
    cap_add:
      - NET_ADMIN                  # Required for the optional iptables firewall (init-firewall.sh)
    volumes:
      # Project directories — mount your host projects into the container
      - ~/code/my-project:/home/claude/projects/my-project
      - ~/code/another-project:/home/claude/projects/another-project
      # Config — share your Claude auth/settings so you don't need to re-authenticate
      - ~/.claude:/home/claude/.claude
      - ~/.claude.json:/home/claude/.claude/.claude.json
```

Build and start the container:

```bash
cd docker
docker compose build && docker compose up -d
```

Attach to the running container:

```bash
docker exec -it claude_code /bin/zsh
```
