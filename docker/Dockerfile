# Pull base image.
FROM jlesage/baseimage-gui:debian-11-v4

# Set the name of the application.
ENV APP_NAME="StardewValley"

RUN apt-get update
RUN apt-get install -y wget dpkg unzip tar strace mono-complete xterm gettext-base jq netcat procps libfaudio0 \
    libusb-1.0-0 libc-bin libc6 locales

RUN wget -O box64.deb https://github.com/ryanfortner/box64-debs/raw/master/debian/box64-lx2160a_0.2.7%2B20240321.1dca97b-1_arm64.deb &&\
    dpkg -i box64.deb &&\
    rm box64.deb

RUN wget -O libssl.deb http://ftp.de.debian.org/debian/pool/main/o/openssl/libssl1.1_1.1.1n-0+deb10u3_arm64.deb &&\
    dpkg -i libssl.deb &&\
    rm -f libssl.deb 

ENV TEMP_DIR=/data/temp \
    DEB_FILE=libicu.deb
    
RUN mkdir -p $TEMP_DIR &&\
    cd $TEMP_DIR &&\
    wget -O $DEB_FILE http://ftp.us.debian.org/debian/pool/main/i/icu/libicu67_67.1-7_amd64.deb &&\
    ar -x $DEB_FILE &&\
    tar -xf data.tar.xz &&\
    cp -r $TEMP_DIR/usr/lib/* /data &&\
    rm -rf $TEMP_DIR 

COPY gl4es/ /data/gl4es/

# Game + ModLoader 1.6.2 4.0.1
RUN mkdir -p /data/Stardew && \
    mkdir -p /data/nexus && \
    wget https://mirror.cloudcraft.info/Stardew_Valley_latest.tar.gz -qO /data/latest.tar.gz && \
    tar xf /data/latest.tar.gz -C /data/Stardew && \
    rm /data/latest.tar.gz 

RUN wget -qO dotnet.tar.gz https://download.visualstudio.microsoft.com/download/pr/d4b71fac-a2fd-4516-ac58-100fb09d796a/e79d6c2a8040b59bf49c0d167ae70a7b/dotnet-sdk-5.0.408-linux-arm64.tar.gz &&\
     tar -zxf dotnet.tar.gz -C /usr/share/dotnet &&\
     rm dotnet.tar.gz &&\
     ln -s /usr/share/dotnet/dotnet /usr/bin/dotnet 

RUN wget https://mirror.cloudcraft.info/SMAPI_latest.zip -qO /data/nexus.zip && \
    unzip /data/nexus.zip -d /data/nexus/ && \
    SMAPI_INSTALLER=$(find /data/nexus -name 'SMAPI*.installer' -type f | head -n 1) && \
    /bin/bash -c "SMAPI_NO_TERMINAL=true SMAPI_USE_CURRENT_SHELL=true echo -e "2\n\n" | $SMAPI_INSTALLER --install --game-path "/data/Stardew/game"" || :

# Add Mods & Scripts
COPY ["mods/", "/data/Stardew/game/Mods/"]
COPY scripts/ /opt/

RUN chmod +x /data/Stardew/game/StardewValley && \
    chmod -R 777 /data/Stardew/ && \
    chown -R 1000:1000 /data/Stardew && \
    chmod +x /opt/*.sh

RUN mkdir /etc/services.d/utils && touch /etc/services.d/app/utils.dep
COPY run /etc/services.d/utils/run 
RUN chmod +x /etc/services.d/utils/run 

COPY docker-entrypoint.sh /startapp.sh
RUN chmod +x /startapp.sh