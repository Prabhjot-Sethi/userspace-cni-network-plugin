# To build:
#  docker build --rm -t dpdk-app-centos .
#


# -------- Builder stage.
FROM centos
MAINTAINER Billy McFall <bmcfall@redhat.com>

#
# Install required packages
#
RUN rpm --import https://mirror.go-repo.io/centos/RPM-GPG-KEY-GO-REPO && curl -s https://mirror.go-repo.io/centos/go-repo.repo | tee /etc/yum.repos.d/go-repo.repo
RUN yum groupinstall -y "Development Tools"
RUN yum install -y wget numactl-devel git golang make; yum clean all

#
# Download and Build APP-NetUtil
#
WORKDIR /root/go/src/
RUN go get github.com/openshift/app-netutil 2>&1 > /tmp/UserspaceDockerBuild.log || echo "Can ignore no GO files."
WORKDIR /root/go/src/github.com/openshift/app-netutil
RUN make c_sample
RUN cp bin/libnetutil_api.so /lib64/libnetutil_api.so; cp bin/libnetutil_api.h /usr/include/libnetutil_api.h

#
# Download and Build DPDK
#
ENV DPDK_VER 19.08
ENV DPDK_DIR /usr/src/dpdk-${DPDK_VER}
WORKDIR /usr/src/
RUN wget http://fast.dpdk.org/rel/dpdk-${DPDK_VER}.tar.xz
RUN tar -xpvf dpdk-${DPDK_VER}.tar.xz

ENV RTE_TARGET=x86_64-native-linuxapp-gcc
ENV RTE_SDK=${DPDK_DIR}
WORKDIR ${DPDK_DIR}
# DPDK_VER 19.08
RUN sed -i -e 's/EAL_IGB_UIO=y/EAL_IGB_UIO=n/' config/common_linux
RUN sed -i -e 's/KNI_KMOD=y/KNI_KMOD=n/' config/common_linux
RUN sed -i -e 's/LIBRTE_KNI=y/LIBRTE_KNI=n/' config/common_linux
RUN sed -i -e 's/LIBRTE_PMD_KNI=y/LIBRTE_PMD_KNI=n/' config/common_linux
# DPDK_VER 19.02
#RUN sed -i -e 's/EAL_IGB_UIO=y/EAL_IGB_UIO=n/' config/common_linuxapp
#RUN sed -i -e 's/KNI_KMOD=y/KNI_KMOD=n/' config/common_linuxapp
#RUN sed -i -e 's/LIBRTE_KNI=y/LIBRTE_KNI=n/' config/common_linuxapp
#RUN sed -i -e 's/LIBRTE_PMD_KNI=y/LIBRTE_PMD_KNI=n/' config/common_linuxapp
RUN make install T=${RTE_TARGET} DESTDIR=${RTE_SDK}

#
# Build TestPmd
#
#WORKDIR ${DPDK_DIR}/app/test-pmd
#COPY ./dpdk-args.c ./dpdk-args.c
#COPY ./dpdk-args.h ./dpdk-args.h
#COPY ./testpmd_eal_init.txt ./testpmd_eal_init.txt
#COPY ./testpmd_launch_args_parse.txt ./testpmd_launch_args_parse.txt
#COPY ./testpmd_substitute.sh ./testpmd_substitute.sh
#RUN ./testpmd_substitute.sh
#RUN make
#RUN cp testpmd /usr/bin/dpdk-app

#
# Build l3fwd
#
WORKDIR ${DPDK_DIR}/examples/l3fwd
COPY ./dpdk-args.c ./dpdk-args.c
COPY ./dpdk-args.h ./dpdk-args.h
COPY ./l3fwd_eal_init.txt ./l3fwd_eal_init.txt
COPY ./l3fwd_parse_args.txt ./l3fwd_parse_args.txt
COPY ./l3fwd_substitute.sh ./l3fwd_substitute.sh
RUN ./l3fwd_substitute.sh
RUN make
RUN cp build/l3fwd /usr/bin/dpdk-app

# -------- Import stage.
# Docker 17.05 or higher
##FROM centos

# Install UserSpace CNI
##COPY --from=0 /usr/bin/dpdk-app /usr/bin/dpdk-app

COPY ./docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["dpdk-app"]
