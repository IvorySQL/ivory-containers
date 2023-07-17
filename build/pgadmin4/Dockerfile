ARG BASEOS
ARG BASEVER
ARG PG_FULL
ARG PREFIX
FROM highgo-base:${BASEOS}-${PG_FULL}-${BASEVER}

# For RHEL8 all arguments used in main code has to be specified after FROM
ARG BASEOS
ARG PACKAGER
ARG PG_MAJOR
ARG PGADMIN_VER
ARG IVY_MAJOR

RUN rpm -i https://ftp.postgresql.org/pub/pgadmin/pgadmin4/yum/pgadmin4-redhat-repo-2-1.noarch.rpm \
        && ${PACKAGER} -y update epel-release \
        && ${PACKAGER} -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm \
        && ${PACKAGER} -y makecache

LABEL name="pgadmin4" \
        summary="pgAdmin4 GUI utility" \
        description="Provides GUI for the pgAdmin 4 utility." \
        io.k8s.description="pgadmin4 container" \
        io.k8s.display-name="pgAdmin 4" \
        io.openshift.tags="postgresql,postgres,monitoring,pgadmin4,pgadmin,database,crunchy"

RUN mkdir -p /opt/crunchy/bin /opt/crunchy/conf /var/lib/pgadmin /var/log/pgadmin \
        /certs /run/httpd

RUN ${PACKAGER} -y install --nodocs \
                --setopt=skip_missing_names_on_install=False \
                mod_ssl \
                python3-gssapi \
                openssl \
                sqlite \
                make \
                epel-release \
                soci-postgresql-devel \
                krb5-devel \
                wget \
                python3-devel \
                sqlite-devel \
                gcc \
                --setopt=obsoletes=0 pgadmin4-7.4 \
                postgresql${PG_MAJOR//.}-server \
        && ${PACKAGER} -y clean all 

COPY requirements.txt /
COPY my-cert.pem /
COPY Setup /
RUN wget https://www.python.org/ftp/python/3.9.9/Python-3.9.9.tgz \
    && yum install -y openssl-devel \
    && export OPENSSL_LIBS=/usr/lib64/libssl.so \
    && tar zxvf Python-3.9.9.tgz \
    && dnf -y install gcc zlib* libffi-devel\
    && cd Python-3.9.9/ \
    && sed -i 's/PKG_CONFIG openssl /PKG_CONFIG openssl11 /g' configure \
    && rm ./Modules/Setup \
    && cp /Setup ./Modules/Setup \
    && ./configure --prefix=/usr/local/python3 --with-openssl=/usr/lib64/libssl.so --with-openssl-rpath=/usr/lib64 --enable-optimizations \
    && make && make install \
    && rm -rf /usr/bin/python3 \
    && ln -s /usr/local/python3/bin/python3 /usr/bin/python3 \
    && rm -rf /usr/bin/pip3 \
    && ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3 \
    && export PIP_CERT=/my-cert.pem \
    && pip3 config set global.trusted-host \
    "pypi.org files.pythonhosted.org pypi.python.org" \
    --trusted-host=pypi.python.org \
    --trusted-host=pypi.org \
    --trusted-host=files.pythonhosted.org \
    && pip3 install --no-cache-dir -r /requirements.txt --trusted-host pypi.org --trusted-host files.pythonhosted.org

RUN cp -rp /usr/pgadmin4/web /usr/local/python3/lib/python3.9/site-packages/pgadmin4-web \
        && chown -R 2:0 usr/local/python3/lib/python3.9/site-packages/pgadmin4-web \
                /var/lib/pgadmin /var/log/pgadmin /certs /etc/httpd /run/httpd \
                /var/log/httpd \
        && chmod -R g=u /usr/local/python3/lib/python3.9/site-packages/pgadmin4-web \
                /var/lib/pgadmin /var/log/pgadmin /certs /etc/httpd /run/httpd /var/log/httpd \
        && ln -sf /var/lib/pgadmin/config_local.py /usr/local/python3/lib/python3.9/site-packages/pgadmin4-web/config_local.py \
        && ln -sf /var/lib/pgadmin/pgadmin.conf /etc/httpd/conf.d/pgadmin.conf \
        && alternatives --set python /usr/bin/python3 



# Preserving PGVERSION out of paranoia
ENV PGROOT="/usr/pgsql-${PG_MAJOR}" PGVERSION="${PG_MAJOR}"

ADD bin/pgadmin4/ /opt/crunchy/bin
ADD bin/common /opt/crunchy/bin
ADD conf/pgadmin4/ /opt/crunchy/conf

RUN cp /opt/crunchy/conf/httpd.conf /etc/httpd/conf/httpd.conf \
        && rm /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/ssl.conf

EXPOSE 5050

# The VOLUME directive must appear after all RUN directives to ensure the proper
# volume permissions are applied when building the image
VOLUME ["/var/lib/pgadmin", "/certs", "/run/httpd"]

# Defines a unique directory name that will be utilized by the nss_wrapper in the UID script
ENV NSS_WRAPPER_SUBDIR="pgadmin"

ENTRYPOINT ["opt/crunchy/bin/uid_daemon.sh"]

USER 2

CMD ["/opt/crunchy/bin/start-pgadmin4.sh"]
