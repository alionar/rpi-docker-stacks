ARG JUPYTER_DOCKER_IMAGE_VERSION
ARG JUPYTER_DOCKER_IMAGE
FROM $JUPYTER_DOCKER_IMAGE:$JUPYTER_DOCKER_IMAGE_VERSION

USER root

# Install packages related to Jupyterlab components
COPY jupyterlab-extensions.txt jupyterlab-extensions.txt
RUN pip install -r ./jupyterlab-extensions.txt

USER ${NB_UID}

# Install python packages
# RUN pip install $HOME/req.txt