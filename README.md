# tensorflow-ppc64-doc
How to build tensorflow from source on POWER.

**The next steps were tested only in Ubuntu 16.10 _without_ GPU support**

To build Tensorflow from source first we have to build bazel (v0.45).
Bazel, in its turn, also need its dependencies to be built from source. They are protobuf (v3.0.0) and grpc-java plugin (v1.0.0).

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
git clone https://github.com/tensorflow/tensorflow.git
cd tensorflow
# if you are using gcc-6, run sed -i "s/march=native/mcpu=native/g" configure first
./configure
bazel build --config=opt //tensorflow/tools/pip_package:build_pip_package
```
