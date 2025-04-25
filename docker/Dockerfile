# This Source Code Form is subject to the terms of the Mozilla Public
# License, v. 2.0. If a copy of the MPL was not distributed with this
# file, You can obtain one at http://mozilla.org/MPL/2.0/.
#
# Copyright (c) 2021-present Kaleidos INC

FROM python:3.11-slim
LABEL maintainer="support@taiga.io"

ENV DEBIAN_FRONTEND=noninteractive \
    PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONFAULTHANDLER=1 \
    DJANGO_SETTINGS_MODULE=settings.config \
    PATH="/opt/venv/bin:$PATH"

# Create a virtualenv
RUN python -m venv /opt/venv

# System dependencies (incl. libxml2 & libxslt)
RUN apt-get update && apt-get install -y \
    build-essential \
    libffi-dev \
    libssl-dev \
    libjpeg-dev \
    zlib1g-dev \
    libpq5 \
    libpq-dev \
    python3-dev \
    libxml2-dev \
    libxslt1-dev \
    gettext \
    git \
    net-tools \
    procps \
    wget \
    gnupg \
    ca-certificates

# Install Python tooling
RUN pip install --upgrade pip setuptools wheel

# Copy project code
WORKDIR /taiga-back
COPY . /taiga-back

# Install dependencies
RUN pip install --verbose -r requirements.txt && \
    pip install --verbose -r requirements-contribs.txt

# App preparation: compile messages, collect static, permission handling
RUN bash -c "\
    python manage.py compilemessages && \
    python manage.py collectstatic --no-input && \
    chmod +x docker/entrypoint.sh docker/async_entrypoint.sh && \
    cp docker/config.py settings/config.py && \
    groupadd --system taiga --gid=999 && \
    useradd --system --no-create-home --gid taiga --uid=999 --shell=/bin/bash taiga && \
    mkdir -p /taiga-back/media/exports && \
    chown -R taiga:taiga /taiga-back \
"

# Clean up to reduce image size
RUN apt-get purge -y \
    build-essential \
    gettext \
    git \
    libpq-dev \
    net-tools \
    procps \
    wget && \
    apt-get autoremove -y && \
    rm -rf /var/lib/apt/lists/* /root/.cache && \
    rm requirements.txt requirements-contribs.txt && \
    find . -name '__pycache__' -exec rm -r '{}' + && \
    find . -name '*pyc' -exec rm -r '{}' + && \
    find . -name '*po' -exec rm -r '{}' +

EXPOSE 8000
ENTRYPOINT ["./docker/entrypoint.sh"]

