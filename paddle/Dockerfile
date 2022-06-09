#FROM artifacts.iflytek.com/docker-private/atp/paddlepaddle/paddle:2.2.2-zhjt
#FROM  paddlepaddle/paddle:2.2.2-gpu-cuda10.2-cudnn7
FROM paddlepaddle/paddle:2.3.0-gpu-cuda11.2-cudnn8

LABEL maintainer="AICLOUD ATP <iflytek.com>"


COPY ./atp-odeon /usr/local/bin/atp-odeon
COPY ./requirements.txt /requirements.txt

RUN pip install -r /requirements.txt -i https://pypi.mirrors.ustc.edu.cn/simple/  



ARG NB_USER="atp"
ARG NB_UID="1000"
ARG NB_GID="100"

# Fix DL4006
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

USER root
COPY sources.list /etc/apt
RUN rm -rf /etc/apt/sources.list.d



ENV DEBIAN_FRONTEND noninteractive
RUN apt-get update && \
    apt-get install --yes  \
    tini \
    wget \
    ca-certificates \
    build-essential \ 
    sudo \
    cmake \
    locales \
    fonts-liberation \
    run-one && \
    apt-get clean && rm -rf /var/lib/apt/lists/* && \
    echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen
RUN  mkdir /etc/apt/sources.list.d/ && curl -sL https://deb.nodesource.com/setup_14.x  | bash -
RUN apt-get install -y nodejs

# Configure environment
ENV CONDA_DIR=/opt/conda \
    SHELL=/bin/bash \
    NB_USER="${NB_USER}" \
    NB_UID=${NB_UID} \
    NB_GID=${NB_GID} \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8
ENV PATH="${CONDA_DIR}/bin:${PATH}" \
    HOME="/home/${NB_USER}"
    #HOME="/work"

RUN mkdir "${HOME}"

# Copy a script that we will use to correct permissions after running certain commands
COPY fix-permissions /usr/local/bin/fix-permissions
RUN chmod a+rx /usr/local/bin/fix-permissions

# Enable prompt color in the skeleton .bashrc before creating the default NB_USER
# hadolint ignore=SC2016
RUN sed -i 's/^#force_color_prompt=yes/force_color_prompt=yes/' /etc/skel/.bashrc && \
   # Add call to conda init script see https://stackoverflow.com/a/58081608/4413446
   echo 'eval "$(command conda shell.bash hook 2> /dev/null)"' >> /etc/skel/.bashrc

# Create NB_USER with name jovyan user with UID=1000 and in the 'users' group
# and make sure these dirs are writable by the `users` group.
RUN echo "auth requisite pam_deny.so" >> /etc/pam.d/su && \
    sed -i.bak -e 's/^%admin/#%admin/' /etc/sudoers && \
    sed -i.bak -e 's/^%sudo/#%sudo/' /etc/sudoers && \
    useradd -l -m -s /bin/bash -N -u "${NB_UID}" "${NB_USER}" && \
    mkdir -p "${CONDA_DIR}" && \
    chown "${NB_USER}:${NB_GID}" "${CONDA_DIR}" && \
    chmod g+w /etc/passwd && \
    fix-permissions "${HOME}" && \
    fix-permissions "${CONDA_DIR}"
RUN  fix-permissions "${HOME}"


#RUN pip install paddleocr==2.5 -i https://pypi.tuna.tsinghua.edu.cn/simple some-package
RUN pip install notebook jupyterlab -i https://pypi.tuna.tsinghua.edu.cn/simple some-package


#RUN pip install ipykernel -i https://pypi.tuna.tsinghua.edu.cn/simple some-package && \
RUN pip install  ipykernel ipython -i https://pypi.tuna.tsinghua.edu.cn/simple some-package
RUN python -m ipykernel install --user --name paddle2.2.2 --display-name "paddle2.3.0"

EXPOSE 8888

# Configure container startup
ENTRYPOINT ["tini", "-g", "--"]
CMD ["start-notebook.sh"]

# Copy local files as late as possible to avoid cache busting
COPY start.sh start-notebook.sh start-singleuser.sh /usr/local/bin/
# Currently need to have both jupyter_notebook_config and jupyter_server_config to support classic and lab
COPY jupyter_notebook_config.py /etc/jupyter/

# Fix permissions on /etc/jupyter as root
USER root

RUN mkdir -p /work
RUN chmod 755 /work && \
    chmod 777 -R "${HOME}" && \
    chmod 777 -R "${HOME}/.local" && \
    rm -rf /etc/bash.bashrc

ENV HOME="/home/atp"
RUN mkdir -p ${HOME}
RUN fix-permissions ${HOME}


# Prepare upgrade to JupyterLab V3.0 #1205
RUN sed -re "s/c.NotebookApp/c.ServerApp/g" \
    /etc/jupyter/jupyter_notebook_config.py > /etc/jupyter/jupyter_server_config.py && \
    fix-permissions /etc/jupyter/ && \
    rm -rf /usr/local/bin/wget /usr/local/bin/df /usr/local/bin/lsblk /usr/local/bin/mount /usr/local/bin/umount

USER root

# install nodejs

RUN pip install jupyterlab-lsp 'python-lsp-server[all]' -i https://pypi.tuna.tsinghua.edu.cn/simple some-package
RUN pip install --upgrade jupyterlab jupyterlab-git jupyterlab-github -i https://pypi.tuna.tsinghua.edu.cn/simple


RUN jupyter labextension install @jupyterlab/toc
ENV PATH=/home/atp/.local/bin:${PATH}

#WORKDIR "${HOME}/work"
WORKDIR "/work"
