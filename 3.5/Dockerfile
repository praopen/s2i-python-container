# This image provides a Python 3.5 environment you can use to run your Python
# applications.
FROM centos/s2i-base-centos7

EXPOSE 8080

ENV PYTHON_VERSION=3.5 \
    PATH=$HOME/.local/bin/:$PATH \
    PYTHONUNBUFFERED=1 \
    PYTHONIOENCODING=UTF-8 \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    PIP_NO_CACHE_DIR=off

ENV SUMMARY="Platform for building and running Python $PYTHON_VERSION applications" \
    DESCRIPTION="Python $PYTHON_VERSION available as docker container is a base platform for \
building and running various Python $PYTHON_VERSION applications and frameworks. \
Python is an easy to learn, powerful programming language. It has efficient high-level \
data structures and a simple but effective approach to object-oriented programming. \
Python's elegant syntax and dynamic typing, together with its interpreted nature, \
make it an ideal language for scripting and rapid application development in many areas \
on most platforms."

LABEL summary="$SUMMARY" \
      description="$DESCRIPTION" \
      io.k8s.description="$DESCRIPTION" \
      io.k8s.display-name="Python 3.5" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="builder,python,python35,rh-python35" \
      com.redhat.component="python35-docker" \
      name="centos/python-35-centos7" \
      version="3.5" \
      usage="s2i build https://github.com/sclorg/s2i-python-container.git --context-dir=3.5/test/setup-test-app/ centos/python-35-centos7 python-sample-app" \
      maintainer="SoftwareCollections.org <sclorg@redhat.com>"

RUN yum install -y centos-release-scl-rh && \
    yum-config-manager --enable centos-sclo-rh-testing && \
    INSTALL_PKGS="rh-python35 rh-python35-python-devel rh-python35-python-setuptools rh-python35-python-pip \
	 nss_wrapper httpd24 httpd24-httpd-devel httpd24-mod_ssl httpd24-mod_auth_kerb httpd24-mod_ldap \
         httpd24-mod_session atlas-devel gcc-gfortran libffi-devel libtool-ltdl enchant" && \
    yum install -y --setopt=tsflags=nodocs --enablerepo=centosplus $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
    # Remove centos-logos (httpd dependency, ~20MB of graphics) to keep image
    # size smaller.
    rpm -e --nodeps centos-logos && \
    yum clean all -y

RUN yum install libproj-devel binutils make geos-devel
RUN wget http://download.osgeo.org/gdal/1.11.2/gdal-1.11.2.tar.gz && \
    tar xzf gdal-1.11.2.tar.gz && \
    cd gdal-1.11.2 && \
    ./configure && \
    make && \
    make install && \
    cd .. && \
    rm -rf gdal*

RUN wget http://download.osgeo.org/geos/geos-3.4.2.tar.bz2 && \
    tar xjf geos-3.4.2.tar.bz2 && \
    cd geos-3.4.2 && \
    ./configure && \
    make && \
    make install && \
    cd .. && \
    rm -rf geos*

RUN wget http://download.osgeo.org/proj/proj-4.9.3.tar.gz \
    && tar -xvf proj-4.9.3.tar.gz \
    && cd proj-4.9.3 \
    && ./configure \
    && make -j4\
    && make install && \
    cd .. && \
    rm -rf proj*

RUN ldconfig
    

RUN cp -r /usr/local/lib/* /opt/rh/rh-python35/root/usr/lib64/.
RUN yum install -y gcc binutils
#RUN yum install -y epel-release && yum install -y postgis2_94
#RUN pip install Shapely==1.6.3
# Copy the S2I scripts from the specific language image to $STI_SCRIPTS_PATH.
COPY ./s2i/bin/ $STI_SCRIPTS_PATH

# Copy extra files to the image.
COPY ./root/ /

# - Create a Python virtual environment for use by any application to avoid
#   potential conflicts with Python packages preinstalled in the main Python
#   installation.
# - In order to drop the root user, we have to make some directories world
#   writable as OpenShift default security model is to run the container
#   under random UID.
RUN source scl_source enable rh-python35 && \
    virtualenv ${APP_ROOT} && \
    chown -R 1001:0 ${APP_ROOT} && \
    fix-permissions ${APP_ROOT} && \
    rpm-file-permissions && \
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib

USER 1001

# Set the default CMD to print the usage of the language image.
CMD $STI_SCRIPTS_PATH/usage
