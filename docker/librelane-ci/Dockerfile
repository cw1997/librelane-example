# CI image: Nix + LibreLane checkout + nix-shell warmup (smoke test pulls toolchain/PDK into /nix/store).
# Aligns with https://librelane.readthedocs.io/en/latest/installation/nix_installation/installation_linux.html
# and newcomers tutorial smoke test flags.

FROM nixos/nix:2.24.15

# Official image is a minimal layered rootfs (no apk). It already ships bash + CA bundle; add git from nixpkgs.
# FOSSI cache + flakes (see LibreLane Linux Nix install docs).
RUN mkdir -p /etc/nix \
  && printf '%s\n' \
    'extra-substituters = https://nix-cache.fossi-foundation.org' \
    'extra-trusted-public-keys = nix-cache.fossi-foundation.org:3+K59iFwXqKsL7BNu6Guy0v+uTlwsxYQxjspXzqLYQs=' \
    'extra-experimental-features = nix-command flakes' \
    'sandbox = false' \
    >> /etc/nix/nix.conf \
  && nix profile install nixpkgs#git --accept-flake-config

ARG LIBRELANE_REF=main
ENV LIBRELANE_ROOT=/librelane

# Shallow clone: branch or tag name in LIBRELANE_REF; for arbitrary SHAs set ref then fetch (second RUN).
RUN set -eux; \
  if git clone --depth 1 --branch "${LIBRELANE_REF}" https://github.com/librelane/librelane "${LIBRELANE_ROOT}"; then \
    :; \
  else \
    rm -rf "${LIBRELANE_ROOT}"; \
    git clone https://github.com/librelane/librelane "${LIBRELANE_ROOT}"; \
    git -C "${LIBRELANE_ROOT}" fetch --depth 1 origin "${LIBRELANE_REF}"; \
    git -C "${LIBRELANE_ROOT}" checkout FETCH_HEAD; \
  fi

WORKDIR ${LIBRELANE_ROOT}

# Warm closure + sky130 via upstream smoke test (large download; intended for cached CI runs).
RUN nix-shell --run "librelane --log-level ERROR --condensed --show-progress-bar --smoke-test"
