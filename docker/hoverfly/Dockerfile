FROM fedora:latest

ENV PACKAGE_LIST less unzip
ENV HOVERFLY_DOWNLOAD_URI https://github.com/SpectoLabs/hoverfly/releases/download/v0.14.2/hoverfly_bundle_linux_amd64.zip
ENV HOME /home/hoverfly

RUN dnf -y install $PACKAGE_LIST && \
    dnf clean all && \
    curl -o /tmp/hoverfly.zip -L $HOVERFLY_DOWNLOAD_URI && \
    unzip /tmp/hoverfly.zip -d /tmp/hoverfly && \
    mv /tmp/hoverfly/hover* /usr/bin && \
    rm -r /tmp/hoverfly /tmp/hoverfly.zip && \
    mkdir -m 775 $HOME && \
    chmod 775 /etc/passwd

WORKDIR $HOME

ADD ./s2i /usr/libexec/s2i

USER 1001

EXPOSE 8500 8888

CMD ["/usr/libexec/s2i/run"]
