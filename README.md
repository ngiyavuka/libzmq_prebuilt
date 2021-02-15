# Prebuilt libzmq build artifacts.

Ideally, we'd build libzmq from source in bazel. As a quick way of compiling and linking against libzmq without taking the time to write bazel build rules, here are some prebuilt artifacts that can be used as dependencies.

## How to use these in a repo.

In your WORKSPACE file, add:

```
//TODO(mike).
```

## To prebuild for more architectures

* Clone libzmq source to your target from https://github.com/zeromq/libzmq.
* Build with whatever commands are appropriate. In its simplest form, this will be a `./configure && make`
* Copy `src/.libs/libzmq.so` into a directory in this repo that described the build.
