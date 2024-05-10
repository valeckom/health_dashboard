ARG DEBIAN_VERSION

FROM debian:${DEBIAN_VERSION}

ARG FLUTTER_VERSION
ARG JAVA_VERSION
ARG ANDROID_SDK
ARG ANDROID_BUILD_TOOLS_VERSION
ARG ANDROID_COMMANDLINE_TOOLS

RUN apt-get -q update

# Dockerfile dependencies
RUN apt-get install -qq \
    curl \
    unzip \
    wget

# JAVA
RUN apt-get -qq install openjdk-${JAVA_VERSION}-jdk-headless && \
    java -version

ENV JAVA_HOME /usr/lib/jvm/java-${JAVA_VERSION}-openjdk-amd64

# Android SDK
# https://developer.android.com/studio/#downloads
ENV ANDROID_SDK_URL="https://dl.google.com/android/repository/commandlinetools-linux-${ANDROID_COMMANDLINE_TOOLS}_latest.zip" \
    ANDROID_SDK_ROOT="/opt/android" \
    ANDROID_HOME="/opt/android"

ENV PATH $PATH:$ANDROID_SDK_ROOT/cmdline-tools/latest/bin:$ANDROID_SDK_ROOT/platform-tools:$ANDROID_SDK_ROOT/build-tools/$ANDROID_BUILD_TOOLS_VERSION
# ANDROID_HOME/ANDROID_SDK_ROOT must be defined for cordova/capcito
RUN echo "export ANDROID_SDK_ROOT=${ANDROID_SDK_ROOT}" >> ~/.bashrc && \
    echo "export ANDROID_HOME=${ANDROID_SDK_ROOT}" >> ~/.bashrc

RUN apt-get install -qq usbutils

WORKDIR /opt

RUN mkdir android && cd android && \
    wget -O tools.zip ${ANDROID_SDK_URL} && \
    unzip tools.zip && rm tools.zip && \
    cd cmdline-tools && \
    mkdir latest && \
    ls | grep -v latest | xargs mv -t latest

RUN mkdir /root/.android && touch /root/.android/repositories.cfg && \
    while true; do echo 'y'; sleep 2; done | sdkmanager "platform-tools" "build-tools;$ANDROID_BUILD_TOOLS_VERSION" && \
    while true; do echo 'y'; sleep 2; done | sdkmanager "platforms;android-$ANDROID_SDK"

RUN chmod a+x -R $ANDROID_SDK_ROOT && \
    chown -R root:root $ANDROID_SDK_ROOT && \
    rm -rf /opt/android/licenses

# SSH
RUN apt-get install -qq openssh-server && \
    sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config && \
    echo "root:0000" | chpasswd

# Flutter
ENV FLUTTER_HOME="/usr/bin/flutter"
ENV PATH $PATH:$FLUTTER_HOME/bin

RUN apt-get install -qq xz-utils zip libglu1-mesa && \
    mkdir /usr/bin/flutter && \
    curl -O https://storage.googleapis.com/flutter_infra_release/releases/stable/linux/flutter_linux_${FLUTTER_VERSION}-stable.tar.xz && \
    tar -xf flutter_linux_${FLUTTER_VERSION}-stable.tar.xz -C /usr/bin && \
    rm flutter_linux_${FLUTTER_VERSION}-stable.tar.xz && \
    echo 'export PATH="/usr/bin/flutter/bin:$PATH"' >> ~/.bash_profile

# Jetbrains devcontainer dependencies
#RUN apt-get install -qq procps

# Development dependencties
RUN apt-get install -qq git
#    aspell

# FIX: Git command returns fatal error: "detected dubious ownership"
RUN git config --system --add safe.directory '*';

# Update non-login terminal's path
RUN echo "export PATH=$PATH:/etc/profile" >> ~/.bashrc

# Clean up
RUN rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    apt-get autoremove -y && \
    apt-get clean

ENTRYPOINT \
    # Start the SSH server
    service ssh start && \
    # start the ADB server listening on all ports.
    # This allows us to see the device in the host's Chrome debugger
    adb -a server && \
    # Keep the container up
    tail -f /dev/null
