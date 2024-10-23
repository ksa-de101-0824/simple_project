# Base image
FROM python:3.11.8-slim

# Metadata
LABEL maintainer="Adam"
LABEL version="1.0.0"
LABEL description="DE Airflow Container"

# Environment variables
ENV DEBIAN_FRONTEND=noninteractive \
    TERM=linux \
    AIRFLOW_VERSION=2.8.2 \
    AIRFLOW_USER_HOME=/usr/local/airflow \
    AIRFLOW_HOME=/usr/local/airflow \
    LANGUAGE=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    LC_ALL=en_US.UTF-8 \
    LC_CTYPE=en_US.UTF-8 \
    LC_MESSAGES=en_US.UTF-8

# Arguments
ARG AIRFLOW_DEPS=""
ARG PYTHON_DEPS=""

# Install dependencies and set up environment
RUN set -ex \
    && apt-get update -yqq \
    && apt-get upgrade -yqq \
    && apt-get install -yqq --no-install-recommends \
        freetds-dev \
        libkrb5-dev \
        libsasl2-dev \
        libssl-dev \
        libffi-dev \
        libpq-dev \
        git \
        freetds-bin \
        build-essential \
        default-libmysqlclient-dev \
        apt-utils \
        curl \
        rsync \
        netcat-traditional \
        pkg-config \
        locales \
    && sed -i 's/^# en_US.UTF-8 UTF-8$/en_US.UTF-8 UTF-8/g' /etc/locale.gen \
    && locale-gen \
    && update-locale LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8 \
    && useradd -ms /bin/bash -d ${AIRFLOW_USER_HOME} airflow \
    && pip install -U pip setuptools wheel \
    && pip install pytz pyOpenSSL ndg-httpsclient pyasn1 python-dotenv requests \
    && pip install apache-airflow[crypto,celery,postgres,hive,jdbc,mysql,papermill,snowflake,ssh${AIRFLOW_DEPS:+,}${AIRFLOW_DEPS}]==${AIRFLOW_VERSION} \
    && pip install deltalake 'redis==3.2' \
    && if [ -n "${PYTHON_DEPS}" ]; then pip install ${PYTHON_DEPS}; fi \
    && apt-get purge --auto-remove -yqq freetds-dev libkrb5-dev libsasl2-dev libssl-dev libffi-dev libpq-dev git \
    && apt-get autoremove -yqq --purge \
    && apt-get clean \
    && rm -rf \
        /var/lib/apt/lists/* \
        /tmp/* \
        /var/tmp/* \
        /usr/share/man \
        /usr/share/doc \
        /usr/share/doc-base

# Copy configuration files
COPY script/entrypoint.sh /entrypoint.sh
COPY config/airflow.cfg ${AIRFLOW_USER_HOME}/airflow.cfg

# Set permissions
RUN chown -R airflow: ${AIRFLOW_USER_HOME}

# Expose ports ##if not working readd 8080
EXPOSE 5555 8793

# Set user and working directory
USER airflow
WORKDIR ${AIRFLOW_USER_HOME}

# Set entrypoint and default command
ENTRYPOINT ["/entrypoint.sh"]
CMD ["webserver"]