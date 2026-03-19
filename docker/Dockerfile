# syntax=docker/dockerfile:1
# Copyright 2023 Google. All rights reserved.
#
# Use of this source code is governed by a BSD-style license that can be
# found in the LICENSE file.

FROM ubuntu:22.04

ARG INSTALL_TARGET_DIR
ARG HOST_TOOL_ARCHIVE_NAME
ARG VIVADO_VERSION

ENV XILINX_INSTALL_LOCATION=$INSTALL_TARGET_DIR
ENV DEBIAN_FRONTEND=noninteractive

RUN echo "dash dash/sh boolean false" | debconf-set-selections
RUN DEBIAN_FRONTEND=$DEBIAN_FRONTEND dpkg-reconfigure dash

# Single-layer apt-get install to reduce image size.
# Credit: This single-layer optimization was contributed by @gretel in PR #5.
RUN apt-get update && apt-get install -y \
		autoconf \
		build-essential \
		chrpath \
		cpio \
		diffstat \
		expect \
		fakeroot \
		gawk \
		gcc-multilib \
		git \
		gzip \
		less \
		libglib2.0-dev \
		libgtk2.0-0 \
		libgtk2.0-dev \
		libncurses5-dev \
		libsdl1.2-dev \
		libtinfo5 \
		libtool \
		libtool-bin \
		locales \
		locales-all \
		lsb-release \
		make \
		net-tools \
		pax \
		python3-gi \
		rsync \
		screen \
		socat \
		tar \
		xorg \
		xterm \
		xvfb \
		zlib1g-dev \
		zlib1g-dev \
	  	bison \
	  	flex \
	  	gnupg \
	  	iproute2 \
	  	libselinux1 \
	  	libssl-dev \
	  	texinfo \
	  	tftpd \
	  	tofrodos \
	  	unzip \
	  	update-inetd \
	  	wget \
		dos2unix \
	 	google-perftools \
		default-jre \
	&& rm -rf /var/lib/apt/lists/*

# TODO: filmil - there is a more canonical way to set the locale.
RUN env LANG=en_US.UTF-8 locale-gen --purge en_US.UTF-8 \
	&& echo -e 'LANG="en_US.UTF-8"\nLANGUAGE="en_US:en"\n' > /etc/default/locale

# Copy only the small config file (not the ~96 GB archive!)
COPY install_config.txt /tmp/install_config.txt
COPY wi_authentication_key /root/.Xilinx/wi_authentication_key

# Extract archive and install Vivado in a SINGLE layer using bind mount.
# The archive is mounted read-only from the build context — never copied
# into an image layer. Extraction + install + cleanup happen atomically,
# so only the final installed files persist.
# Credit: This bind-mount optimization was contributed by @gretel in PR #5.
RUN --mount=type=bind,source=${HOST_TOOL_ARCHIVE_NAME},target=/tmp/installer.bin \
    mkdir -p /tmp/vivado-extract \
    && cd /tmp/vivado-extract/ \
    && /tmp/installer.bin -- \
        --config /tmp/install_config.txt \
        --batch Install \
        --location $XILINX_INSTALL_LOCATION \
        --agree XilinxEULA,3rdPartyEULA \
    && echo "Cleaning up extracted archive..." \
    && cd && rm -rf /tmp/vivado-extract /tmp/install_config.txt /root/.Xilinx/wi_authentication_key

VOLUME /src
VOLUME /work

COPY run.sh /
RUN chmod a+x run.sh

WORKDIR /work
