FROM ubuntu:16.04
LABEL maintainer="info@msav.ru"

ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update -y \
    && apt-get -y --no-install-recommends install \
        bzip2 \
        locales \
        wget \
        gnupg2 \
        rename \
        net-tools \
        gosu \
        nano \
        mc \
    && localedef -i ru_RU -c -f UTF-8 -A /usr/share/locale/locale.alias ru_RU.UTF-8

ENV LANG ru_RU.utf8

ENV PGSQL_MAJOR 9.6
ENV PGSQL_VERSION ${PGSQL_MAJOR}.7-1
ENV PGSQL_DIST ./dist
ENV PGSQL_DEST /tmp/pgsql

ADD ${PGSQL_DIST} /opt/

RUN mkdir ${PGSQL_DEST} \
    && cp /opt/postgresql* ${PGSQL_DEST} \
    && find ${PGSQL_DEST} -type f -exec rename -v 'y/_/\-/' {} \;

RUN if [ ! -f ${PGSQL_DEST}/postgresql-${PGSQL_VERSION}.1C-amd64-deb.tar.bz2 ]; then \
        echo "File '${PGSQL_DEST}/postgresql_${PGSQL_VERSION}.1C_amd64_deb.tar.bz2' not found."; \
        exit 1; \
    fi

RUN cd ${PGSQL_DEST} \
    && tar -xvjf postgresql-${PGSQL_VERSION}.1C-amd64-deb.tar.bz2 \
    && rm postgresql-${PGSQL_VERSION}.1C-amd64-deb.tar.bz2 \
    && find ${PGSQL_DEST} -type f -iname '*.deb' -exec mv {} ${PGSQL_DEST} \; \
    && find ${PGSQL_DEST} -type f -iname '*.deb' -exec rename -v 'y/_/\-/' {} \; \
    && apt-get -y --no-install-recommends install \
        libgssapi-krb5-2 \
        libldap-2.4-2 \
        postgresql-client-common \
        libedit2 \
        ssl-cert \
        tzdata \
        libxml2 \
        postgresql-common \
    && dpkg -i libpq5-${PGSQL_VERSION}.1C-amd64.deb \
    && dpkg -i postgresql-client-9.6-${PGSQL_VERSION}.1C-amd64.deb \
    && dpkg -i postgresql-9.6-${PGSQL_VERSION}.1C-amd64.deb

# RUN sed -ri /local\\s+all\\s+postgres\\s+/s/peer/trust/ /etc/postgresql/9.6/main/pg_hba.conf
RUN sed -ri s/^shared_preload_libraries/#shared_preload_libraries/ /etc/postgresql/9.6/main/postgresql.conf

ENV PATH /usr/lib/postgresql/${PGSQL_MAJOR}/bin:$PATH
ENV PGDATA /var/lib/postgresql/data

COPY ./entrypoint.sh /
RUN chmod 775 /entrypoint.sh

ENTRYPOINT [ "/entrypoint.sh" ]
#CMD [ "/entrypoint.sh" ]

EXPOSE 5432
    