ARG BASE=quay.io/centos-bootc/centos-bootc:stream10
ARG KVER=6.12.0-184.el10

FROM ${BASE} AS build
ARG KVER
RUN dnf config-manager --set-enabled crb
RUN dnf install -y rpmdevtools createrepo_c
RUN dnf download --source kernel-${KVER}
RUN rpm --install       ./kernel-${KVER}.src.rpm
RUN dnf builddep -y     ./kernel-${KVER}.src.rpm

RUN sed -i 's|# CONFIG_PM_DEVFREQ is not set|CONFIG_PM_DEVFREQ=y|' \
  /root/rpmbuild/SOURCES/kernel-aarch64-rhel.config
RUN sed -i 's|# CONFIG_ARM_TEGRA_DEVFREQ is not set|CONFIG_ARM_TEGRA_DEVFREQ=y|' \
  /root/rpmbuild/SOURCES/kernel-aarch64-rhel.config
RUN cat <<EOF >> /root/rpmbuild/SOURCES/kernel-aarch64-rhel.config
CONFIG_DEVFREQ_THERMAL=y
CONFIG_DEVFREQ_GOV_SIMPLE_ONDEMAND=y
CONFIG_DEVFREQ_GOV_PERFORMANCE=y
CONFIG_DEVFREQ_GOV_POWERSAVE=y
CONFIG_DEVFREQ_GOV_USERSPACE=m
CONFIG_DEVFREQ_GOV_PASSIVE=y
CONFIG_PM_DEVFREQ_EVENT=y
EOF

RUN rpmbuild --without automotive --without realtime --without debug \
  --without arm64_64k --without realtime_arm64_64k --without debug \
  -bb /root/rpmbuild/SPECS/kernel.spec

RUN createrepo /root/rpmbuild/RPMS

FROM ${BASE} AS base
ARG KVER
RUN --mount=from=build,source=/root/rpmbuild/RPMS,target=/kernel \
  dnf install -y \
  /kernel/aarch64/kernel-${KVER}.aarch64.rpm \
  /kernel/aarch64/kernel-core-${KVER}.aarch64.rpm \
  /kernel/aarch64/kernel-modules-${KVER}.aarch64.rpm \
  /kernel/aarch64/kernel-modules-core-${KVER}.aarch64.rpm \
  /kernel/aarch64/kernel-modules-extra-${KVER}.aarch64.rpm \
  /kernel/aarch64/kernel-modules-extra-matched-${KVER}.aarch64.rpm

FROM base AS devel
ARG KVER
RUN --mount=from=build,source=/root/rpmbuild/RPMS,target=/kernel \
  dnf install -y \
  /kernel/aarch64/kernel-devel-${KVER}.aarch64.rpm \
  /kernel/aarch64/kernel-devel-matched-${KVER}.aarch64.rpm \
  /kernel/aarch64/kernel-headers-${KVER}.aarch64.rpm
