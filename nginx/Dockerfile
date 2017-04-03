FROM fedora:24

ENV container=docker

RUN dnf -y --setopt=tsflags=nodocs install \
        nginx \
    && dnf -y clean all

RUN systemctl enable nginx

EXPOSE 80

VOLUME [ "/run", "/tmp", "/usr/share/nginx/html" ]

CMD [ "/usr/sbin/init" ]

