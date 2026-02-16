ARG BASE=quay.io/centos-bootc/centos-bootc:stream10
ARG KVER=6.12.0-204.el10

FROM ${BASE} AS build
ARG KVER
ADD cs.repo        /etc/yum.repos.d/centos.repo
ADD cs-source.repo /etc/yum.repos.d/centos-addons.repo
RUN dnf config-manager --set-enabled crb
RUN dnf install -y rpmdevtools createrepo_c
RUN dnf download --source kernel-${KVER}
RUN rpm --install       ./kernel-${KVER}.src.rpm
RUN dnf builddep -y     ./kernel-${KVER}.src.rpm --nobest

COPY kernel.patch /root/rpmbuild/SOURCES/patch-6.12-redhat.patch

RUN rpmbuild --without automotive --without realtime --without debug \
  --without arm64_64k --without realtime_arm64_64k --without debug \
  -bb /root/rpmbuild/SPECS/kernel.spec

RUN createrepo /root/rpmbuild/RPMS

FROM ${BASE} AS base
ARG KVER
ADD cs.repo        /etc/yum.repos.d/centos.repo
ADD cs-source.repo /etc/yum.repos.d/centos-addons.repo
RUN --mount=from=build,source=/root/rpmbuild/RPMS,target=/kernel \
  dnf reinstall -y \
  /kernel/aarch64/kernel-${KVER}.aarch64.rpm \
  /kernel/aarch64/kernel-core-${KVER}.aarch64.rpm \
  /kernel/aarch64/kernel-modules-${KVER}.aarch64.rpm \
  /kernel/aarch64/kernel-modules-core-${KVER}.aarch64.rpm \
  /kernel/aarch64/kernel-modules-extra-${KVER}.aarch64.rpm \
  /kernel/aarch64/kernel-modules-extra-matched-${KVER}.aarch64.rpm

FROM base AS devel
ARG KVER
RUN --mount=from=build,source=/root/rpmbuild/RPMS,target=/kernel \
  dnf reinstall -y \
  /kernel/aarch64/kernel-devel-${KVER}.aarch64.rpm \
  /kernel/aarch64/kernel-devel-matched-${KVER}.aarch64.rpm \
  /kernel/aarch64/kernel-headers-${KVER}.aarch64.rpm
