FROM ubuntu

ARG UID=1000

ENV PROJECT_HOME /home/patchwork/patchwork

ENV db_user root
ENV db_pass password

ENV DJANGO_SETTINGS_MODULE patchwork.settings.dev
ENV DEBIAN_FRONTEND noninteractive
ENV PYTHONUNBUFFERED 1
ENV https_proxy=10.239.4.160:913
ENV http_proxy=10.239.4.160:913

# System
# python3.4-minimal libpython3.4-stdlib libpython3.4-minimal \
# findutils is for python3.4... :/
COPY tools/docker/trusty.list /etc/apt/sources.list.d/trusty.list
COPY ./hosts /etc/hosts
COPY ./apt.conf /etc/apt/apt.conf

RUN cat /etc/hosts /etc/resolv.conf /etc/apt/apt.conf

RUN https_proxy=10.239.4.160:913 \
    http_proxy=10.239.4.160:913 \
    apt-get update -qq && \
    apt-get install -y --no-install-recommends --allow-downgrades \
    python-dev python-pip python-setuptools python-wheel \
    python3.5-dev python3-pip python3-setuptools python3-wheel \
    python3.4-dev findutils=4.4.2-7 sudo vim\
    libmysqlclient-dev mysql-client curl unzip xvfb chromium-chromedriver \
    chromium-browser build-essential && \
    ln -s /usr/lib/chromium-browser/chromedriver /usr/bin/

# User
RUN useradd --uid=$UID --create-home patchwork


# Python requirements.
# If you update requirements, you should rebuild the container.
# entrypoint.sh will prompt you to do this.
# we install both Python 2 and Python 3 versions so you can use either
COPY requirements-*.txt /tmp/
RUN https_proxy=10.239.4.160:913 http_proxy=10.239.4.160:913
RUN pip3 install virtualenv tox && \
    pip3 install -r /tmp/requirements-dev.txt
RUN pip2 install virtualenv tox && \
    pip2 install -r /tmp/requirements-dev.txt
# we deliberately leave the requirements files in tmp so we can
# ping the user in entrypoint.sh if the change them!

COPY tools/docker/bashrc /tmp/bashrc

# we put the code in ~/patchwork rather than ~ so that we
# can put in these bashrc snippets without dirtying the
# working directory
#RUN python3 manage.py createsupersuer

RUN cat /tmp/bashrc >> /home/patchwork/.bashrc

COPY tools/docker/entrypoint.sh /usr/local/bin/entrypoint.sh

ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
# Give sudo access to patchwork
RUN adduser patchwork sudo
COPY tools/docker/sudoers /tmp/sudoers
RUN cat /tmp/sudoers >> /etc/sudoers
USER patchwork
WORKDIR /home/patchwork/patchwork