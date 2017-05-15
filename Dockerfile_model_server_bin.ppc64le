FROM ppc64le/ubuntu:16.04

MAINTAINER Abhishek Dasgupta <abdasgupta@in.ibm.com>

RUN apt-get update && apt-get install -y \
        autoconf \
        libtool \
        build-essential \
        curl \
        git \
        libfreetype6-dev \
        libpng12-dev \
        libzmq3-dev \
        pkg-config \
        python-dev \
        python-numpy \
        python-pip \
        software-properties-common \
        swig \
        zip \
        zlib1g-dev \
        libcurl3-dev \
        openjdk-8-jdk \
        openjdk-8-jre-headless \
        libblas-dev \
        liblapack-dev \
        libatlas-base-dev \
        gfortran \
        && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

RUN curl -fSsL -O https://bootstrap.pypa.io/get-pip.py && \
    python get-pip.py && \
    rm get-pip.py

RUN pip install grpcio

RUN pip --no-cache-dir install \
        ipykernel \
        jupyter \
        matplotlib \
        numpy \
        scipy \
        sklearn \
        pandas \
        && \
python -m ipykernel.kernelspec

# Set up our notebook config.
COPY jupyter_notebook_config.py /root/.jupyter/

# Jupyter has issues with being run directly:
#   https://github.com/ipython/ipython/issues/7062
# We just add a little wrapper script.
COPY run_jupyter.sh /

RUN update-ca-certificates -f

RUN git clone -b v3.1.0 https://github.com/abdasgupta/protobuf-ppc64le.git /protobuf && \
  cd /protobuf && \
  ./autogen.sh && \
  ./configure --prefix=/opt/DL/protobuf && \
  make && \
  make install && \
  cd / && \
  rm -rf /protobuf

ENV PROTOC /opt/DL/protobuf/bin/protoc
ENV CXXFLAGS -I/opt/DL/protobuf/include
ENV LDFLAGS -L/opt/DL/protobuf/lib/

RUN git clone -b ppc-grpc https://github.com/abdasgupta/grpc-java-ppc64le.git /grpc-java && \
   cd /grpc-java && \
   ./gradlew build --info -Pprotoc=$PROTOC -PprotoInclude="/opt/DL/protobuf/include/" -PprotoLib="/opt/DL/protobuf/lib" --continue || true && \
   mkdir /opt/DL/grpc-java && \
   cp compiler/build/artifacts/java_plugin/protoc-gen-grpc-java.exe /opt/DL/grpc-java && \
   cd / && \
   rm -rf /grpc-java

ENV GRPC_JAVA_PLUGIN=/opt/DL/grpc-java/protoc-gen-grpc-java.exe

RUN git clone -b ppc-bazel https://github.com/abdasgupta/bazel-ppc64le.git /bazel && \
    cd /bazel && \
    echo "build --spawn_strategy=standalone --genrule_strategy=standalone" >>/tmp/.bazelrc && \
    BAZELRC=/tmp/bazelrc ./compile.sh && \
    cp output/bazel  /usr/bin/ && \
    cd / && \
    rm -rf /root/.cache /bazel

ENV CI_BUILD_PYTHON python

RUN git clone -b ppc-tensorflow-serving --recurse-submodules https://github.com/abdasgupta/tensorflow-serving-ppc64le /tensorflow-serving && \
    cd /tensorflow-serving/tensorflow && \
    tensorflow/tools/ci_build/builds/configured CPU && \
    cd .. && \
    bazel build -j `nproc` --ram_utilization_factor 50 -c opt tensorflow_serving/... && \
    cp bazel-bin/tensorflow_serving/model_servers/tensorflow_model_server / && \
    rm -rf /root/.cache && \
    # Provided support to install prediction service from tensorflow serving as a wheel file,so
    # that user can write her own inception client without building tensorflow serving again.
    bazel build //tensorflow_serving/example:build-pip-package

# To build the wheel file for inception-export so that inception-export.py can run standalone
RUN git clone https://github.com/abdasgupta/tf-automated-serving.git /tf-automated-serving && \
    cd /tf-automated-serving/inception && \
    bazel build //inception:build-pip-package

CMD ["echo Model Server Built"]
