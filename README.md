# tensorflow-ppc64-doc
How to build tensorflow from source on POWER

To build Tensorflow from source first we have to build bazel.
Bazel, in its turn, also need its dependencies to be built from source. They are protobuf (v3.0.0) and grpc-java plugin (v1.0.0).

## Building protobuf
```
git clone https://github.com/google/protobuf.git
cd protobuf
git checkout v3.0.0
git cherry-pick 1760feb621a913189b90fe8595fffb74bce84598 # an external dependecy URL has changed
./autogen.sh && ./configure && make
```

## Building grpc-java
```
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
```
export PROTOC=</path/to/protobuf>/src/protoc
export GRPC_JAVA_PLUGIN=<path/to/grpc-java>/compiler/build/exe/java_plugin/protoc-gen-grpc-java
./compile.sh
```
