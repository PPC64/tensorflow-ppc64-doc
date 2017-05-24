# tensorflow-ppc64-doc
How to build tensorflow from source on POWER.

**The next steps were tested only in Ubuntu 16.10 _without_ GPU support**

As of commit https://github.com/tensorflow/tensorflow/commit/efe5376f3dec8fcc2bf3299a4ff4df6ad3591c88 it is not possible to build Tensorflow from source using https://github.com/ibmsoe/bazel.git since this fork does not support bazel v0.4.5

To build Tensorflow from source first we have to build bazel (v0.4.5.).
Bazel, in its turn, also needs its dependencies to be built from source. They are protobuf (v3.0.0) and grpc-java plugin (v1.0.0).

## Building protobuf
```bash
git clone https://github.com/google/protobuf.git
cd protobuf
git checkout v3.0.0
git cherry-pick 1760feb621a913189b90fe8595fffb74bce84598 # an external dependecy URL has changed
./autogen.sh && ./configure && make
```

## Building grpc-java
```bash
git clone https://github.com/grpc/grpc-java.git
cd grpc-java
git checkout v1.0.0
export CXXFLAGS="-I<path/to/protobuf>/src" LDFLAGS="-L<path/to/protobuf>/src/.libs"
git cherry-pick 862157a84be602c1cabfb46958511489337bfd36  # This commit has Power specific changes
cd compiler
GRPC_BUILD_CMD="../gradlew java_pluginExecutable"
eval $GRPC_BUILD_CMD
```
## Building bazel
```bash
git clone https://github.com/bazelbuild/bazel.git
cd bazel
export PROTOC=</path/to/protobuf>/src/protoc
export GRPC_JAVA_PLUGIN=<path/to/grpc-java>/compiler/build/exe/java_plugin/protoc-gen-grpc-java
./compile.sh
cd output
export PATH=$(pwd):$PATH
```
## Building tensorflow
```bash
sudo apt-get install python-numpy python-dev python-pip python-wheel
git clone https://github.com/PPC64/tensorflow.git
cd tensorflow
./configure
bazel build --config=opt //tensorflow/tools/pip_package:build_pip_package
bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg
sudo pip install /tmp/tensorflow_pkg/tensorflow*.whl
```

Accordingly to the tensorflow's installation tutorial:
> By default, building TensorFlow from sources consumes a lot of RAM.
> If RAM is an issue on your system, you may limit RAM usage by specifying --local_resources 2048,.5,1.0 while invoking bazel.
