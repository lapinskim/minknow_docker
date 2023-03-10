## -*- dockerfile-image-name: "ont-minknow" -*-
# syntax=docker/dockerfile:1
FROM ubuntu:20.04

ENV TZ=Europe/Warsaw

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    gnupg2 && \
    curl -fsSL https://cdn.oxfordnanoportal.com/apt/ont-repo.pub | apt-key add - && \
    echo "deb http://cdn.oxfordnanoportal.com/apt focal-stable non-free" \
    >/etc/apt/sources.list.d/nanoporetech.sources.list && \
    apt-get purge --autoremove -y curl \
    && rm -rf /var/lib/apt/lists/*

ARG MINKNOW_NC_VERSION=22.12.7
ARG MINKNOW_CORE_VERSION=5.4.3-focal
ARG ONT_BREAM4_VERSION=7.4.8-1~focal
ARG ONT_CONFIGURATION_CUSTOMER_VERSION=5.4.7-1~focal
ARG ONT_VBZ_HDF_PLUGIN_VERSION=1.0.7-1~focal

RUN apt-get update && apt-get install -y --no-install-recommends \
    minknow-core-minion-nc="$MINKNOW_CORE_VERSION" \
    ont-bream4-minion="$ONT_BREAM4_VERSION" \
    ont-configuration-customer-minion="$ONT_CONFIGURATION_CUSTOMER_VERSION" \
    ont-vbz-hdf-plugin="$ONT_VBZ_HDF_PLUGIN_VERSION" \
    && rm -rf /var/lib/apt/lists/*

ARG DISTRO_VERSION="$MINKNOW_NC_VERSION"
ARG DISTRO_STATUS="stable"
ARG MINKNOW_DIRECTORY="/opt/ont/minknow/"

RUN cd "${MINKNOW_DIRECTORY}" || exit 1; \
    ./bin/config_editor --set distribution_version="${DISTRO_VERSION}" \
    --set distribution_status="${DISTRO_STATUS}" \
    --conf system --filename=conf/sys_conf

RUN ln -s /bin/true /bin/systemctl

EXPOSE 8000-8007/tcp
EXPOSE 9501-9505/tcp

ENTRYPOINT ["/opt/ont/minknow/bin/mk_manager_svc"]
CMD ["-d",\
    "--print-rpc-ports"]
