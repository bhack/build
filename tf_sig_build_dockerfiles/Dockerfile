################################################################################
FROM ubuntu:20.04 as builder
################################################################################

# Install devtoolset build dependencies
COPY setup.packages.sh setup.packages.sh
COPY builder.packages.txt builder.packages.txt
RUN /setup.packages.sh /builder.packages.txt

# Install devtoolset-7 in /dt7 with gclibc 2.12 and libstdc++ 4.4, for building
# manylinux2010-compatible packages. Scripts expect to be in the root directory.
COPY builder.devtoolset/fixlinks.sh /fixlinks.sh
COPY builder.devtoolset/rpm-patch.sh /rpm-patch.sh
COPY builder.devtoolset/build_devtoolset.sh /build_devtoolset.sh
RUN /build_devtoolset.sh devtoolset-7 /dt7 

################################################################################
FROM nvidia/cuda:11.2.1-base-ubuntu20.04 as devel
################################################################################

COPY --from=builder /dt7 /dt7

# Install required development packages but delete unneeded CUDA bloat
# CUDA must be cleaned up in the same command to prevent Docker layer bloating
COPY setup.sources.sh /setup.sources.sh
COPY setup.packages.sh /setup.packages.sh
COPY setup.cuda.sh /setup.cuda.sh
COPY devel.packages.txt /devel.packages.txt
RUN /setup.sources.sh && /setup.packages.sh /devel.packages.txt && /setup.cuda.sh

# Install various tools.
# - bats: bash unit testing framework
# - bazelisk: always use the correct bazel version
# - buildifier: clean bazel build deps
# - buildozer: clean bazel build deps
RUN git clone --branch v1.3.0 https://github.com/bats-core/bats-core.git && bats-core/install.sh /usr/local && rm -rf bats-core
RUN wget https://github.com/bazelbuild/bazelisk/releases/download/v1.7.4/bazelisk-linux-amd64 -O /usr/local/bin/bazel && chmod +x /usr/local/bin/bazel
RUN wget https://github.com/bazelbuild/buildtools/releases/download/3.5.0/buildifier -O /usr/local/bin/buildifier && chmod +x /usr/local/bin/buildifier
RUN wget https://github.com/bazelbuild/buildtools/releases/download/3.5.0/buildozer -O /usr/local/bin/buildozer && chmod +x /usr/local/bin/buildozer

# Setup Python environment. PYTHON_VERSION is e.g. "python3.8"
ARG PYTHON_VERSION
COPY setup.python.sh /setup.python.sh
COPY devel.requirements.txt /devel.requirements.txt
RUN /setup.python.sh $PYTHON_VERSION devel.requirements.txt

# Setup build and environment
COPY devel.usertools /usertools
COPY devel.bashrc /root/.bashrc

