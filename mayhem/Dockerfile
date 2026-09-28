# syntax=docker/dockerfile:1.7
#
# Commit-image Dockerfile (SMALL form) for s2geometry — builds on the org C/C++ base.
FROM ghcr.io/savantenvs/base:20260621

ARG COMMIT
ARG REPO_URL
# SOURCE_DATE_EPOCH stays an ARG (available to build.sh); do NOT bake it as an ENV — an
# empty value breaks clang both here and for any in-image rebuild. build.sh unsets it when empty.
ARG SOURCE_DATE_EPOCH

# Sanitizer override: the base exports SANITIZER_FLAGS (ASan+UBSan, halting) as the default.
# Declared WITHOUT a default so the base ENV flows through when not passed.
ARG SANITIZER_FLAGS
ARG COVERAGE_FLAGS=
ARG MAYHEM_JOBS=

# Air-gap the deps (SPEC §6.5): at UPSTREAM (2022-05-28) CMakeLists.txt has no FetchContent —
# it does find_package(absl REQUIRED) and add_subdirectory(${GTEST_ROOT} ...), the same recipe
# the mayhemheroes fork's own build.sh used (git-clone abseil HEAD + `make install`, GTEST_ROOT).
# Pre-fetch era-appropriate, C++11-compatible pins instead: abseil 20220623.1 (the LTS current
# when this bug was found) and googletest release-1.11.0; build.sh builds+installs absl offline
# and points GTEST_ROOT at the googletest source so this re-runs fully air-gapped.
USER root
RUN set -ex \
 && mkdir -p /opt/deps && cd /opt/deps \
 && curl -fsSL -o absl.zip https://github.com/abseil/abseil-cpp/archive/refs/tags/20220623.1.zip \
 && echo "54707f411cb62a26a776dad5fd60829098c181700edcd022ea5c2ca49e9b7ef1  absl.zip" | sha256sum -c \
 && unzip -q absl.zip && mv abseil-cpp-20220623.1 absl && rm absl.zip \
 && curl -fsSL -o googletest.zip https://github.com/google/googletest/archive/refs/tags/release-1.11.0.zip \
 && echo "353571c2440176ded91c2de6d6cd88ddd41401d14692ec1f99e35d013feda55a  googletest.zip" | sha256sum -c \
 && unzip -q googletest.zip && mv googletest-release-1.11.0 googletest && rm googletest.zip \
 && chown -R 2000:2000 /opt/deps
USER mayhem

# Source enters from the build CONTEXT (the CI checkout): the FULL repo (incl. .git
# history) is copied to /mayhem as `mayhem`; build.sh builds in place.
WORKDIR /mayhem
COPY --chown=2000:2000 . /mayhem

RUN bash mayhem/build.sh

# Run the project's functional tests at build time (a failing suite fails the image build).
RUN if [ -f mayhem/test.sh ]; then bash mayhem/test.sh; else echo "no mayhem/test.sh — repo ships no functional tests"; fi

LABEL org.opencontainers.image.source="${REPO_URL}" \
      org.opencontainers.image.revision="${COMMIT}" \
      rlenv.tier="commit" \
      rlenv.source_date_epoch="${SOURCE_DATE_EPOCH}" \
      rlenv.user="mayhem"

USER mayhem
WORKDIR /mayhem
