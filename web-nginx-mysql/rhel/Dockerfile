FROM registry.access.redhat.com/ubi8/ubi
MAINTAINER Alexey Pustovalov <alexey.pustovalov@zabbix.com>

ARG DNF_FLAGS_COMMON="-y"
ARG DNF_FLAGS_PERSISTENT="${DNF_FLAGS_COMMON}"
ARG DNF_FLAGS_DEV="${DNF_FLAGS_COMMON}"
ENV TERM=xterm

ARG BUILD_DATE
ARG VCS_REF

ARG MAJOR_VERSION=4.4
ARG RELEASE=7
ARG ZBX_VERSION=${MAJOR_VERSION}.7

ARG ZBX_SOURCES=https://git.zabbix.com/scm/zbx/zabbix.git
ENV ZBX_VERSION=${ZBX_VERSION} ZBX_SOURCES=${ZBX_SOURCES}

LABEL name="zabbix/zabbix-web-mysql" \
      maintainer="alexey.pustovalov@zabbix.com" \
      vendor="Zabbix LLC" \
      version="${MAJOR_VERSION}" \
      release="${RELEASE}" \
      summary="Zabbix web-interface based on Nginx web server with MySQL database support" \
      description="Zabbix web-interface based on Nginx web server with MySQL database support" \
      url="https://www.zabbix.com/" \
      run="docker run --name zabbix-web-nginx --link mysql-server:mysql --link zabbix-server:zabbix-server -p 80:80 -d registry.connect.redhat.com/zabbix/zabbix-web-nginx-44:${ZBX_VERSION}" \
      io.k8s.description="Zabbix web-interface based on Nginx web server with MySQL database support" \
      io.k8s.display-name="Zabbix Frontend (Nginx)" \
      io.openshift.expose-services="http:http,https:https" \
      io.openshift.tags="zabbix,zabbix-web,mysql,nginx" \
      org.label-schema.name="zabbix-web-mysql-rhel" \
      org.label-schema.vendor="Zabbix LLC" \
      org.label-schema.url="https://zabbix.com/" \
      org.label-schema.description="Zabbix web-interface based on Nginx web server with MySQL database support" \
      org.label-schema.vcs-ref="${VCS_REF}" \
      org.label-schema.build-date="${BUILD_DATE}" \
      org.label-schema.schema-version="1.0" \
      org.label-schema.license="GPL v2.0" \
      org.label-schema.usage="https://www.zabbix.com/documentation/${MAJOR_VERSION}/manual/installation/containers" \
      org.label-schema.version="${ZBX_VERSION}" \
      org.label-schema.vcs-url="${ZBX_SOURCES}" \
      org.label-schema.docker.cmd="docker run --name zabbix-web-nginx --link mysql-server:mysql --link zabbix-server:zabbix-server -p 80:80 -d registry.connect.redhat.com/zabbix/zabbix-web-nginx-44:${ZBX_VERSION}"

STOPSIGNAL SIGTERM

COPY ["conf/etc/yum.repo.d/nginx.repo", "/etc/yum.repos.d/nginx.repo"]
            
### add licenses to this directory
COPY ["licenses", "/licenses"]
            
RUN set -eux && INSTALL_PKGS="bash \
            curl \
            mariadb \
            nginx \
            php-bcmath \
            php-fpm \
            php-gd \
            php-ldap \
            php-mbstring \
            python3-setuptools \
            php-mysqlnd \
            php-xml" && \
    dnf ${DNF_FLAGS_COMMON} install https://repo.zabbix.com/zabbix/${MAJOR_VERSION}/rhel/8/x86_64/zabbix-release-${MAJOR_VERSION}-1.el8.noarch.rpm && \
    REPOLIST="rhel-8-for-x86_64-baseos-rpms,rhel-8-for-x86_64-appstream-rpms,zabbix-non-supported,nginx-stable" && \
    dnf ${DNF_FLAGS_COMMON} update-minimal --disablerepo "*" --enablerepo rhel-8-for-x86_64-baseos-rpms --setopt=tsflags=nodocs \
        --security --sec-severity=Important --sec-severity=Critical && \
    dnf ${DNF_FLAGS_COMMON} install --disablerepo "*" --enablerepo "${REPOLIST}" --setopt=tsflags=nodocs ${INSTALL_PKGS} && \
    groupadd --system zabbix && \
    adduser -r --shell /sbin/nologin \
            -g zabbix -G dialout \
            -d /var/lib/zabbix/ \
        zabbix && \
    mkdir -p /etc/zabbix && \
    mkdir -p /etc/zabbix/web && \
    chown --quiet -R zabbix:root /etc/zabbix && \
    rm -f /etc/php-fpm.d/www.conf && \
    mkdir -p /var/lib/php/ && \
    chown --quiet -R nginx:nginx /var/lib/php/ && \
    easy_install-3.6 --install-dir /usr/lib/python3.6/site-packages/ supervisor && \
    mkdir -p /etc/supervisor/conf.d/ && \
    dnf ${DNF_FLAGS_COMMON} clean all && \
    rm -rf /var/cache/yum && \
    rm -rf /var/lib/yum/yumdb/* && \
    rm -rf /usr/lib/udev/hwdb.d/*

RUN set -eux && REPOLIST="rhel-8-for-x86_64-baseos-rpms,rhel-8-for-x86_64-appstream-rpms,zabbix-non-supported" && \
    INSTALL_PKGS="gettext \
            git" && \
    dnf ${DNF_FLAGS_COMMON} install --disablerepo "*" --enablerepo "${REPOLIST}" --setopt=tsflags=nodocs ${INSTALL_PKGS} && \
    cd /usr/share/ && \
    git clone ${ZBX_SOURCES} --branch ${ZBX_VERSION} --depth 1 --single-branch zabbix-${ZBX_VERSION} && \
    mkdir /usr/share/zabbix/ && \
    cp -R /usr/share/zabbix-${ZBX_VERSION}/frontends/php/* /usr/share/zabbix/ && \
    rm -rf /usr/share/zabbix-${ZBX_VERSION}/ && \
    cd /usr/share/zabbix/ && \
    rm -f conf/zabbix.conf.php && \
    rm -rf tests && \
    ./locale/make_mo.sh && \
    chown --quiet -R nginx:nginx /usr/share/zabbix && \
    ln -s /usr/lib/python3.6/site-packages/supervisord /usr/bin/supervisord && \
    dnf ${DNF_FLAGS_COMMON} history undo `dnf -q history | sed -n 3p |column -t | cut -d' ' -f1` && \
    dnf ${DNF_FLAGS_COMMON} clean all && \
    rm -rf /var/cache/yum && \
    rm -rf /var/lib/yum/yumdb/* && \
    rm -rf /usr/lib/udev/hwdb.d/* && \
    rm -rf /etc/udev/hwdb.bin && \
    rm -rf /root/.pki

EXPOSE 80/TCP 443/TCP

WORKDIR /usr/share/zabbix

VOLUME ["/etc/ssl/nginx"]

COPY ["conf/etc/supervisor/", "/etc/supervisor/"]
COPY ["conf/etc/zabbix/nginx.conf", "/etc/zabbix/"]
COPY ["conf/etc/zabbix/nginx_ssl.conf", "/etc/zabbix/"]
COPY ["conf/etc/zabbix/web/zabbix.conf.php", "/etc/zabbix/web/"]
COPY ["conf/etc/nginx/nginx.conf", "/etc/nginx/"]
COPY ["conf/etc/php-fpm.conf", "/etc/php-fpm.conf"]
COPY ["conf/etc/php.d/99-zabbix.ini", "/etc/php.d/99-zabbix.ini"]
COPY ["docker-entrypoint.sh", "/usr/bin/"]

ENTRYPOINT ["docker-entrypoint.sh"]
