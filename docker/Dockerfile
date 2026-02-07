FROM debian:bookworm
ARG USERNAME=claude
ARG TZ=America/Los_Angeles
ENV TZ="$TZ"

# Install basic development tools and iptables/ipset
RUN apt-get update && apt-get install -y --no-install-recommends \
  less \
  git \
  procps \
  sudo \
  fzf \
  zsh \
  man-db \
  unzip \
  gnupg2 \
  gh \
  iptables \
  ipset \
  iproute2 \
  dnsutils \
  aggregate \
  jq \
  vim \
  curl \
  wget \
  && apt-get clean && rm -rf /var/lib/apt/lists/*

# Create user and group
RUN groupadd --gid 1000 ${USERNAME} && \
  useradd --uid 1000 --gid ${USERNAME} --shell /bin/zsh --create-home ${USERNAME}

ENV DEVCONTAINER=true
ENV CLAUDE_CONFIG_DIR=/home/${USERNAME}/.claude

# Create workspace and config directories and set permissions
RUN mkdir -p /home/${USERNAME}/.claude && \
  chown -R ${USERNAME}:${USERNAME} /home/${USERNAME}/.claude

WORKDIR /home/${USERNAME}/projects
RUN chown -R ${USERNAME}:${USERNAME} /home/${USERNAME}/projects

ARG GIT_DELTA_VERSION=0.18.2
RUN ARCH=$(dpkg --print-architecture) && \
  wget "https://github.com/dandavison/delta/releases/download/${GIT_DELTA_VERSION}/git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb" && \
  dpkg -i "git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb" && \
  rm "git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb"

# Install Go
ARG GO_VERSION=1.25.6
RUN ARCH=$(dpkg --print-architecture) && \
  wget "https://go.dev/dl/go${GO_VERSION}.linux-${ARCH}.tar.gz" && \
  tar -C /usr/local -xzf "go${GO_VERSION}.linux-${ARCH}.tar.gz" && \
  rm "go${GO_VERSION}.linux-${ARCH}.tar.gz"
ENV PATH=$PATH:/usr/local/go/bin

USER ${USERNAME}

# Set the default shell to zsh rather than sh
ENV SHELL=/bin/zsh

# Set the default editor and visual
ENV EDITOR=vim
ENV VISUAL=vim

# Default powerline10k theme
ARG ZSH_IN_DOCKER_VERSION=1.2.1
RUN sh -c "$(wget -O- https://github.com/deluan/zsh-in-docker/releases/download/v${ZSH_IN_DOCKER_VERSION}/zsh-in-docker.sh)" -- \
  -p git \
  -p fzf \
  -a "source /usr/share/doc/fzf/examples/key-bindings.zsh" \
  -a "source /usr/share/doc/fzf/examples/completion.zsh" \
  -x

# Install Claude
RUN curl -fsSL https://claude.ai/install.sh | zsh

# Copy and set up firewall script
COPY ../init-firewall.sh /usr/local/bin/
USER root
RUN chmod +x /usr/local/bin/init-firewall.sh && \
  echo "${USERNAME} ALL=(root) NOPASSWD: /usr/local/bin/init-firewall.sh" > /etc/sudoers.d/${USERNAME}-firewall && \
  chmod 0440 /etc/sudoers.d/${USERNAME}-firewall

USER ${USERNAME}
