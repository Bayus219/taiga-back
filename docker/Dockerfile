# This Source Code Form is subject to the terms of the Mozilla Public
# License, v. 2.0. If a copy of the MPL was not distributed with this
# file, You can obtain one at http://mozilla.org/MPL/2.0/.
#
# Copyright (c) 2021-present Kaleidos INC

FROM python:3.11-slim
LABEL maintainer="support@taiga.io"

ENV DEBIAN_FRONTEND=noninteractive
ENV PYTHONUNBUFFERED=1
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONFAULTHANDLER=1
ENV PATH="/opt/venv/bin:$PATH"

# 🛠 System dependencies for building lxml and Python packages
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

# 🐍 Set up Python virtual environment
RUN python -m venv /opt/venv
RUN . /opt/venv/bin/activate && pip install --upgrade pip setuptools wheel

# 📁 Set working directory & copy project
WORKDIR /taiga-back
COPY . .

# 📦 Install dependencies
RUN pip install --verbose -r requirements.txt
RUN pip install --verbose -r requirements-contribs.txt

# 🧰 Setup scripts and files
RUN chmod +x docker/entrypoint.sh docker/async_entrypoint.sh && \
    python manage.py compilemessages && \
    python manage.py collectstatic --no-input && \
    cp docker/config.py settings/config.py && \
    groupadd --system taiga --gid=999 && \
    useradd --system --no-create-home --gid taiga --uid=999 --shell=/bin/bash taiga && \
    mkdir -p /taiga-back/media/exports && \
    chown -R taiga:taiga /taiga-back

# 🧹 Clean up build dependencies
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

ENV DJANGO_SETTINGS_MODULE=settings.config

EXPOSE 8000
ENTRYPOINT ["./docker/entrypoint.sh"]


