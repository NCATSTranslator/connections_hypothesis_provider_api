###########
# BUILDER #
###########

# first stage of build to pull repos
FROM ubuntu as intermediate

# set work directory
WORKDIR /usr/src/chp_api

# install git
RUN apt-get update \
	&& apt-get install -y git python3-pip python3-dev dos2unix

# add credentials on build
ARG INTUITION_USER
ARG SSH_PRIVATE_KEY
ARG SSH_PUBLIC_KEY
RUN mkdir -p /root/.ssh/ \
	&& chmod 0700 /root/.ssh \
	&& ssh-keyscan intuition.thayer.dartmouth.edu >> ~/.ssh/known_hosts \
	&& chmod 644 ~/.ssh/known_hosts

# add keys (use in windows powershell)
COPY "${SSH_PRIVATE_KEY}" /root/.ssh/
COPY "${SSH_PUBLIC_KEY}" /root/.ssh/
RUN chmod 600 /root/.ssh/id_rsa \
	&& chmod 600 /root/.ssh/id_rsa.pub

# add keys (use in Linux)
#RUN echo "${SSH_PRIVATE_KEY}" > /root/.ssh/id_rsa \
#	&& echo "${SSH_PUBLIC_KEY}" > /root/.ssh/id_rsa.pub \
#	&& chmod 600 /root/.ssh/id_rsa \
#	&& chmod 600 /root/.ssh/id_rsa.pub

#RUN ssh cyakaboski@intuition.thayer.dartmouth.edu -t -t -v -o StrictHostKeyChecking=no

RUN git clone ssh://"${INTUITION_USER}"@intuition.thayer.dartmouth.edu/home/public/git/modules/PyBKB.git
RUN git clone ssh://"${INTUITION_USER}"@intuition.thayer.dartmouth.edu/home/public/git/projects/ChpData.git
RUN git clone https://github.com/di2ag/chp.git

# install dependencies
COPY ./requirements.txt .
RUN pip3 wheel --no-cache-dir --no-deps --wheel-dir /usr/src/chp_api/wheels -r requirements.txt

# gather pybkb wheel
RUN cd PyBKB && python3 setup.py bdist_wheel && cd dist && cp pybkb-1.0.0-py3-none-any.whl /usr/src/chp_api/wheels

# gather chp wheel
RUN cd bkb-pathway-provider && python3 setup.py bdist_wheel && cd dist && cp chp-1.0.0-py3-none-any.whl /usr/src/chp_api/wheels

# gather chp-data wheel
RUN cd ChpData && python3 setup.py bdist_wheel && cd dist && cp chp_data-0.0.1-py3-none-any.whl /usr/src/chp_api/wheels



#########
# FINAL #
#########

#pull official base image
FROM ubuntu:20.04

# add app user
RUN groupadd chp_api && useradd -ms /bin/bash -g chp_api chp_api

# create the appropriate directories
ENV HOME=/home/chp_api
ENV APP_HOME=/home/chp_api/web
RUN mkdir $APP_HOME
RUN mkdir $APP_HOME/staticfiles
WORKDIR $APP_HOME

# set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1
ENV TZ=America/New_York

# set ARGs
ARG DEBIAN_FRONTEND=noninterative

# install dependencies
RUN apt-get update \
	&& apt-get install -y python3-pip graphviz openmpi-bin libopenmpi-dev build-essential libssl-dev libffi-dev python3-dev vim
RUN apt-get install -y libgraphviz-dev python3-pygraphviz
RUN apt-get install -y libpq-dev
RUN apt-get install -y netcat

# copy repo to new image
COPY --from=intermediate /usr/src/chp_api/wheels /wheels
COPY --from=intermediate /usr/src/chp_api/requirements.txt .
RUN pip3 install --upgrade pip
RUN pip3 install --no-cache /wheels/*

# copy entry point
COPY ./entrypoint.sh .

# copy project
COPY . .

# run entrypoint.sh
ENTRYPOINT ["/usr/src/chp_api/entrypoint.sh"]
