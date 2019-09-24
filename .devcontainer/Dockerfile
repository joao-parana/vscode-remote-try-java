#-----------------------------------------------------------------------------------------
# Copyright (c) Microsoft Corporation. All rights reserved.
# Licensed under the MIT License. See LICENSE in the project root for license information.
#-----------------------------------------------------------------------------------------

FROM maven:3.6-jdk-8

# Or your actual UID, GID on Linux if not the default 1000
ARG USERNAME=parana
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# This ARGSs run the commands
# groupadd --gid 1000 parana && useradd -s /bin/bash --uid 1000 --gid 1000 -m parana

# Configure apt
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update \
    && apt-get -y install --no-install-recommends apt-utils dialog 2>&1 \
    #
    # Install git, process tools, lsb-release (common in install instructions for CLIs)
    && apt-get -y install git procps lsb-release \
    #
    # Allow for a consistant java home location for settings - image is changing over time
    && if [ ! -d "/docker-java-home" ]; then ln -s "${JAVA_HOME}" /docker-java-home; fi \
    #
    # Create a non-root user to use if preferred - see https://aka.ms/vscode-remote/containers/non-root-user.
    && groupadd --gid $USER_GID $USERNAME \
    && useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
    # [Optional] Add sudo support for non-root user
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    #
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*

ENV DEBIAN_FRONTEND=

USER $USERNAME
RUN echo "HOME = $HOME" && mkdir -p $HOME/.ssh && \
    ssh-keygen -q -N "" -t rsa -b 4096 -f $HOME/.ssh/id_rsa && \
    find $HOME

COPY .credentials/id_rsa_for_git /home/$USERNAME/.ssh/id_rsa_for_git

RUN ls -la /home/$USERNAME/.ssh && pwd && mkdir $HOME/bin && echo "$HOME/bin" && \
    echo "## "  >> $HOME/.bashrc && \
    echo "alias ssh-jp='eval \`ssh-agent -s\` && ssh-add -D && ssh-add ~/.ssh/id_rsa_for_git  && ssh -T git@bitbucket.org ; ssh -T git@github.com' " >> $HOME/.bashrc

USER root
RUN chown $USERNAME:$USERNAME /home/$USERNAME/.ssh/*
# passwordless ssh
RUN ssh-keygen -q -N "" -t dsa -f /etc/ssh/ssh_host_dsa_key
RUN ssh-keygen -q -N "" -t rsa -f /etc/ssh/ssh_host_rsa_key
RUN ssh-keygen -q -N "" -t rsa -f /root/.ssh/id_rsa
RUN cp /root/.ssh/id_rsa.pub /root/.ssh/authorized_keys

RUN echo "Execute ssh-jp on Bash shell to add credentials for github"
