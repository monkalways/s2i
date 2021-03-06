FROM dotnet/dotnet-50-runtime-centos7
# This image provides a .NET 5.0 environment you can use to run your .NET
# applications.

ENV PATH=/opt/app-root/src/.local/bin:/opt/app-root/src/bin:/opt/app-root/node_modules/.bin:${PATH} \
    STI_SCRIPTS_PATH=/usr/libexec/s2i

LABEL io.k8s.description="Platform for building and running .NET 5 applications" \
    io.openshift.tags="builder,.net,dotnet,dotnetcore,rh-dotnet50"

# Labels consumed by Red Hat build service
LABEL name="dotnet/dotnet-50-centos7" \
    com.redhat.component="rh-dotnet50-container" \
    version="5.0" \
    release="1" \
    architecture="x86_64"

# Labels consumed by Eclipse JBoss OpenShift plugin
LABEL com.redhat.dev-mode="DEV_MODE:false" \
    com.redhat.deployments-dir="/opt/app-root/src"

# Switch to root for package installs
USER 0

# Copy the S2I scripts from the specific language image to $STI_SCRIPTS_PATH.
COPY ./s2i/bin/ /usr/libexec/s2i

# Add Microsoft RPM
# RUN rpm -Uvh https://packages.microsoft.com/config/centos/7/packages-microsoft-prod.rpm

# rh-nodejs14-npm rh-nodejs14-nodejs-nodemon not available
RUN yum install -y centos-release-dotnet centos-release-scl-rh
RUN yum install -y --setopt=tsflags=nodocs rh-nodejs10-npm rh-nodejs10-nodejs-nodemon dotnet-sdk-5.0 rsync
RUN rpm -V rh-nodejs10-npm rh-nodejs10-nodejs-nodemon dotnet-sdk-5.0 rsync
RUN yum clean all -y && \
    # yum cache files may still exist (and quite large in size)
    rm -rf /var/cache/yum/*

# Directory with the sources is set as the working directory.
RUN mkdir /opt/app-root/src
WORKDIR /opt/app-root/src

# Trigger first time actions.
RUN dotnet help

# Since $HOME is set to /opt/app-root, the yum install may have created config
# directories (such as ~/.pki/nssdb) there. These will be owned by root and can
# cause actions that work on all of /opt/app-root to fail. So we need to fix
# the permissions on those too.
RUN chown -R 1001:1001 /opt/app-root
RUN chown -R 1001:1001 /usr/libexec/s2i
# fix-permissions /opt/app-root (permission denied)

ENV ENABLED_COLLECTIONS="$ENABLED_COLLECTIONS rh-nodejs14" \
    # Needed for the `dotnet watch` to detect changes in a container.
    DOTNET_USE_POLLING_FILE_WATCHER=true

# Run container by default as user with id 1001 (default)
USER 1001

# Set the default CMD to print the usage of the language image.
CMD /usr/libexec/s2i/usage
