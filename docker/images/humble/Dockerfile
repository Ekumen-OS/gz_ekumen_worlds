FROM ros:humble-ros-base-jammy AS cacher

WORKDIR /ws/src

COPY . gz-worlds/

RUN mkdir -p /tmp/ws/src \
  && find ./ -name "package.xml" | xargs cp --parents -t /tmp/ws/src \
  && find ./ -name "COLCON_IGNORE" | xargs cp --parents -t /tmp/ws/src \
  || true

FROM ros:humble-ros-base-jammy AS builder

SHELL ["/bin/bash", "-c"]
ENV SHELL=/bin/bash

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
  && apt-get install --no-install-recommends -y \
    ccache \
    curl \
    gdb \
    git \
    gnupg \
    python3-pip \
    mc \
    tmux \
    libglvnd0 \
    libgl1 \
    libglx0 \
    libegl1 \
    libgles2 \
    lsb-release \
    mesa-utils \
  && rm -rf /var/lib/apt/lists/*

ARG GZ_VERSION="harmonic"
RUN curl https://packages.osrfoundation.org/gazebo.gpg --output /usr/share/keyrings/pkgs-osrf-archive-keyring.gpg \
  && echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/pkgs-osrf-archive-keyring.gpg] http://packages.osrfoundation.org/gazebo/ubuntu-stable $(lsb_release -cs) main" > /etc/apt/sources.list.d/gazebo-stable.list \
  && apt-get update && apt-get install --no-install-recommends -y $(if [[ "${GZ_VERSION}" > "fortress" ]]; then echo gz; else echo ignition; fi)-${GZ_VERSION}

RUN pip install pre-commit

ARG USER=developer
ARG GROUP=ekumen

RUN addgroup --gid 1010 $GROUP \
  && adduser --uid 1010 --ingroup $GROUP --home /home/$USER --shell /bin/sh --disabled-password --gecos "" $USER \
  && adduser $USER sudo \
  && adduser $USER dialout \
  && echo "$USER ALL=NOPASSWD: ALL" >> /etc/sudoers.d/$USER

RUN mkdir -p /home/$USER/.gz && chown $USER:$GROUP /home/$USER/.gz

COPY docker/files/fixuid_config.yml /etc/fixuid/config.yml
RUN curl -SsL https://github.com/boxboat/fixuid/releases/download/v0.4/fixuid-0.4-linux-amd64.tar.gz | tar -C /usr/local/bin -xzf - \
  && chmod 4755 /usr/local/bin/fixuid \
  && cd /etc/fixuid \
  && sed -i "s/_USER_/$USER/" config.yml \
  && sed -i "s/_GROUP_/$GROUP/" config.yml

USER $USER:$GROUP

ENV USER_WS=/home/$USER/ws
RUN mkdir -p /$USER_WS

WORKDIR $USER_WS

RUN colcon mixin add default https://raw.githubusercontent.com/colcon/colcon-mixin-repository/master/index.yaml \
  && colcon mixin update default

#
# Install project dependencies

COPY --from=cacher --chown=$USER:$GROUP /tmp/ws/ $USER_WS/
RUN sudo apt-get update \
  && . /opt/ros/humble/setup.sh \
  && rosdep update \
  && rosdep install -i -y --from-path src \
  && sudo rm -rf /var/lib/apt/lists/*

#
# Setup environment

RUN bash -c "echo '. /opt/ros/humble/setup.bash' >> /home/$USER/.bashrc"

ENV WITHIN_DEV=1
ENV MAKEFLAGS="-j 2"

ENTRYPOINT ["fixuid", "-q", "/ros_entrypoint.sh", "/bin/bash"]
