# This file creates a container that runs a jupyter lab server on Raspberry Pi
#
# Originally from: https://github.com/jiwidi/jupyter-lab-docker-rpi/blob/master/Dockerfile

FROM balenalib/raspberry-pi-python:3.7-20200518

ARG NB_USER="jovyan"
ARG NB_UID="1000"
ARG NB_GID="100"

# Fix DL4006
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

USER root

# Install all OS dependencies for notebook server that starts but lacks all
# features (e.g., download as all possible file formats)
ENV DEBIAN_FRONTEND noninteractive
ENV NODE_OPTIONS "--max-old-space-size=2048"
RUN apt-get update \
 && apt-get install -yq --no-install-recommends \
    wget \
    bzip2 \
    ca-certificates \
    sudo \
    locales \
    fonts-liberation \
    run-one \
 && apt-get clean && rm -rf /var/lib/apt/lists/*

RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen

# Configure environment
ENV SHELL=/bin/bash \
    NB_USER=$NB_USER \
    NB_UID=$NB_UID \
    NB_GID=$NB_GID \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8
ENV HOME=/home/$NB_USER

# Copy a script that we will use to correct permissions after running certain commands
COPY fix-permissions /usr/local/bin/fix-permissions
RUN chmod a+rx /usr/local/bin/fix-permissions

# Enable prompt color in the skeleton .bashrc before creating the default NB_USER
RUN sed -i 's/^#force_color_prompt=yes/force_color_prompt=yes/' /etc/skel/.bashrc

# Create NB_USER wtih name jovyan user with UID=1000 and in the 'users' group
# and make sure these dirs are writable by the `users` group.
RUN echo "auth requisite pam_deny.so" >> /etc/pam.d/su && \
    sed -i.bak -e 's/^%admin/#%admin/' /etc/sudoers && \
    sed -i.bak -e 's/^%sudo/#%sudo/' /etc/sudoers && \
    useradd -m -s /bin/bash -N -u $NB_UID $NB_USER && \
    chmod g+w /etc/passwd && \
    fix-permissions $HOME

USER $NB_UID
WORKDIR $HOME

# Setup work directory for backward-compatibility
RUN mkdir /home/$NB_USER/work && \
    fix-permissions /home/$NB_USER

# Install Tini
# Add Tini. Tini operates as a process subreaper for jupyter. This prevents kernel crashes.
ENV TINI_VERSION 0.18.0
ENV CFLAGS="-DPR_SET_CHILD_SUBREAPER=36 -DPR_GET_CHILD_SUBREAPER=37"

ADD https://github.com/krallin/tini/archive/v${TINI_VERSION}.tar.gz /root/v${TINI_VERSION}.tar.gz
RUN apt-get install -y cmake
RUN tar zxvf v${TINI_VERSION}.tar.gz \
        && cd tini-${TINI_VERSION} \
        && cmake . \
        && make \
        && cp tini /usr/bin/. \
        && cd .. \
        && rm -rf "./tini-${TINI_VERSION}" \
        && rm "./v${TINI_VERSION}.tar.gz" \
        && fix-permissions /home/$NB_USER

ENTRYPOINT ["/usr/bin/tini", "--"]

# Install JupyterHub, Lab, Hub
## Update pip and install jupyter
RUN apt-get update && apt-get upgrade
RUN apt-get install -y apt-utils git curl
RUN apt-get install -y libncurses5-dev 
RUN apt-get install -y libzmq-dev 
RUN apt-get install -y libfreetype6-dev 
RUN apt-get install -y libpng-dev
RUN apt-get install -y python-dev --fix-missing 
RUN apt-get install -y patch
RUN apt-get install -y build-essential
RUN apt-get install -y libpq-dev 
RUN apt-get install -y libstdc++6
RUN apt-get install -y libatlas3-base 
RUN apt-get install -y libgfortran5
RUN apt-get install -y libtinfo6
RUN apt-get install -y libncurses6

RUN pip3 install --upgrade pip -i https://www.piwheels.org/simple
RUN pip3 install cython -i https://www.piwheels.org/simple
RUN pip3 install gnureadline ipywidgets jupyter jupyterlab==2.0.0 -i https://www.piwheels.org/simple

RUN wget -O - https://deb.nodesource.com/setup_12.x | bash
RUN apt-get install -y nodejs g++ make software-properties-common libzmq3-dev

RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
RUN apt-get update && apt-get install yarn

# Configure jupyter
RUN jupyter nbextension enable --py widgetsnbextension
RUN jupyter serverextension enable --py jupyterlab
RUN jupyter notebook --generate-config

# Correct permissions
RUN fix-permissions /home/$NB_USER

EXPOSE 8888

# Configure container startup
ENTRYPOINT ["tini", "-g", "--"]
CMD ["start-notebook.sh"]

# Copy local files as late as possible to avoid cache busting
COPY start.sh start-notebook.sh start-singleuser.sh /usr/local/bin/
COPY jupyter_notebook_config.py /etc/jupyter/

# Fix permissions on /etc/jupyter as root
USER root
COPY pip.conf /etc/pip.conf
RUN fix-permissions /etc/jupyter/

# Switch back to jovyan to avoid accidental container runs as root
USER $NB_UID

WORKDIR $HOME

######################################################################

# WORKDIR /root

# # Update pip and install jupyter
# RUN apt-get update && apt-get upgrade
# RUN apt-get install -y apt-utils git curl
# RUN apt-get install -y libncurses5-dev 
# RUN apt-get install -y libzmq-dev 
# RUN apt-get install -y libfreetype6-dev 
# RUN apt-get install -y libpng-dev
# RUN apt-get install -y python-dev --fix-missing 
# RUN apt-get install -y patch
# RUN apt-get install -y build-essential
# RUN apt-get install -y libpq-dev 
# RUN apt-get install -y libstdc++6
# RUN apt-get install -y libatlas3-base 
# RUN apt-get install -y libgfortran5
# RUN apt-get install -y libtinfo6
# RUN apt-get install -y libncurses6

# RUN pip3 install --upgrade pip -i https://www.piwheels.org/simple
# RUN pip3 install cython -i https://www.piwheels.org/simple
# RUN pip3 install gnureadline ipywidgets jupyter jupyterlab==2.0.0 -i https://www.piwheels.org/simple

# RUN wget -O - https://deb.nodesource.com/setup_12.x | bash
# RUN apt-get install -y nodejs g++ make software-properties-common libzmq3-dev

# RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
# RUN echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
# RUN apt-get update && apt-get install yarn

# # Configure jupyter
# RUN jupyter nbextension enable --py widgetsnbextension
# RUN jupyter serverextension enable --py jupyterlab
# RUN jupyter notebook --generate-config
# RUN mkdir notebooks
# #RUN sed -i "/c.NotebookApp.open_browser/c c.NotebookApp.open_browser = False" /root/.jupyter/jupyter_notebook_config.py \
# #        && sed -i "/c.NotebookApp.ip/c c.NotebookApp.ip = '*'" /root/.jupyter/jupyter_notebook_config.py \
# #        && sed -i "/c.NotebookApp.notebook_dir/c c.NotebookApp.notebook_dir = '/root'" /root/.jupyter/jupyter_notebook_config.py \
# #        && sed -i "/c.NotebookApp.password/c c.NotebookApp.password = 'sha1:5815fb7ca805:f09ed218dfcc908acb3e29c3b697079fea37486a'" /root/.jupyter/jupyter_notebook_config.py

# VOLUME /root/notebooks

# # Add Tini. Tini operates as a process subreaper for jupyter. This prevents kernel crashes.
# ENV TINI_VERSION 0.18.0
# ENV CFLAGS="-DPR_SET_CHILD_SUBREAPER=36 -DPR_GET_CHILD_SUBREAPER=37"

# ADD https://github.com/krallin/tini/archive/v${TINI_VERSION}.tar.gz /root/v${TINI_VERSION}.tar.gz
# RUN apt-get install -y cmake
# RUN tar zxvf v${TINI_VERSION}.tar.gz \
#         && cd tini-${TINI_VERSION} \
#         && cmake . \
#         && make \
#         && cp tini /usr/bin/. \
#         && cd .. \
#         && rm -rf "./tini-${TINI_VERSION}" \
#         && rm "./v${TINI_VERSION}.tar.gz"

# ENTRYPOINT ["/usr/bin/tini", "--"]

# EXPOSE 8888

# COPY pip.conf /etc/pip.conf
# RUN apt autoremove
# WORKDIR /root/notebooks

# ENV SHELL=/bin/bash

# CMD ["jupyter", "lab", "--allow-root", "--no-browser"]
