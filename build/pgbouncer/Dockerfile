FROM redhat/ubi8:latest

# For RHEL8 all arguments used in main code has to be specified after FROM
ARG PACKAGER
ARG PG_MAJOR
ARG PGBOUNCER_VER

RUN ${PACKAGER} -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm \
	&& ${PACKAGER} -y makecache

LABEL name="pgbouncer" \
	summary="Lightweight connection pooler for ivory" \
	description="The aim of pgbouncer is to lower the performance impact of opening new connections to PostgreSQL; clients connect through this service. It offers session, transaction and statement pooling." \
	io.k8s.description="pgBouncer" \
	io.k8s.display-name="pgBouncer" \
	io.openshift.tags="postgresql,postgres,pooling,pgbouncer,database,ivory,ivorysql"

RUN ${PACKAGER} -y install --nodocs \
	--disablerepo="epel" \
	pgbouncer-${PGBOUNCER_VER} \
	&& ${PACKAGER} -y clean all

# Preserving PGVERSION out of paranoia
ENV PGVERSION="${PG_MAJOR}"

RUN mkdir -p /opt/crunchy/bin /opt/crunchy/conf /pgconf

ADD bin/pgbouncer /opt/crunchy/bin
ADD bin/common /opt/crunchy/bin
ADD conf/pgbouncer /opt/crunchy/conf

RUN chown -R 2:0 /opt/crunchy /pgconf && \
	chmod -R g=u /opt/crunchy /pgconf

EXPOSE 6432

# The VOLUME directive must appear after all RUN directives to ensure the proper
# volume permissions are applied when building the image
VOLUME ["/pgconf"]

# Defines a unique directory name that will be utilized by the nss_wrapper in the UID script
ENV NSS_WRAPPER_SUBDIR="pgbouncer"

ENTRYPOINT ["opt/crunchy/bin/uid_daemon.sh"]

USER 2

CMD ["/opt/crunchy/bin/start.sh"]
