ARG BASE=quay.io/centos-bootc/centos-bootc:stream10
ARG KVER=6.12.0-150.el10

FROM ${BASE}
ARG KVER
RUN dnf config-manager --set-enabled crb
RUN dnf install -y rpmdevtools
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
COPY kernel.patch /root/rpmbuild/SOURCES/patch-6.12-redhat.patch

RUN rpmbuild --without automotive --without realtime --without debug \
  --without arm64_64k --without realtime_arm64_64k --without debug \
  -bb /root/rpmbuild/SPECS/kernel.spec
