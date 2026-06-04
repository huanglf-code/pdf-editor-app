FROM debian:stable-slim

# Set non-interactive mode for installations
ENV DEBIAN_FRONTEND=noninteractive

# Install apt dependencies
RUN apt-get update && \
    apt-get -qq install -y \
        curl git unzip xz-utils zip libglu1-mesa clang cmake ninja-build mesa-utils libgtk-3-dev \
        libsecret-1-dev libjsoncpp-dev ghostscript webkit2gtk-4.1-dev ca-certificates openjdk-21-jdk \
        wget tar bash && \
    apt-get clean

# Create `nonroot` user
RUN useradd -u 1000 -m nonroot
USER nonroot

# Install Flutter
RUN mkdir -p /home/nonroot/.pub-cache/bin/
ENV FLUTTER_ROOT=/home/nonroot/flutter
ENV PUB_CACHE=/home/nonroot/.pub-cache
ENV PATH="/home/nonroot/flutter/bin:/home/nonroot/.pub-cache/bin:$PATH"
# This ADD command invalidates the Docker cache when a new Flutter release is available
ADD https://raw.githubusercontent.com/flutter/flutter/refs/heads/stable/CHANGELOG.md /tmp/flutter-CHANGELOG.md
RUN git clone https://github.com/flutter/flutter.git /home/nonroot/flutter --branch stable
RUN flutter precache --universal

# Install Android SDK/NDK (pin versions)
ENV ANDROID_SDK_ROOT=/opt/android-sdk \
    ANDROID_HOME=/opt/android-sdk
RUN sudo mkdir -p /opt/android-sdk && sudo chown -R nonroot:nonroot /opt/android-sdk
RUN mkdir -p $ANDROID_SDK_ROOT/cmdline-tools && \
    cd $ANDROID_SDK_ROOT && \
    wget -q https://dl.google.com/android/repository/commandlinetools-linux-11076708_latest.zip -O cmdline-tools.zip && \
    mkdir -p cmdline-tools/latest && \
    unzip -q cmdline-tools.zip -d cmdline-tools/latest && \
    rm cmdline-tools.zip
ENV PATH="$ANDROID_SDK_ROOT/cmdline-tools/latest/bin:$ANDROID_SDK_ROOT/platform-tools:$PATH"
RUN yes | sdkmanager --licenses >/dev/null && \
    sdkmanager --install \
      "platform-tools" \
      "platforms;android-34" \
      "build-tools;34.0.0" \
      "ndk;26.1.10909125"

# Install Rust with targets (using rustup via Flutter image toolchain assumptions)
RUN curl https://sh.rustup.rs -sSf | sh -s -- -y && \
    /home/nonroot/.cargo/bin/rustup target add \
      aarch64-linux-android \
      armv7-linux-androideabi \
      x86_64-linux-android
ENV PATH="/home/nonroot/.cargo/bin:$PATH"

ARG APP_PATH
WORKDIR ${APP_PATH}

# Default command: open a shell
CMD ["/bin/bash"]
