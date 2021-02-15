# Prebuilt libzmq build artifacts.

Ideally, we'd build libzmq from source in bazel. As a quick way of compiling and linking against libzmq without taking the time to write bazel build rules, here are some prebuilt artifacts that can be used as dependencies.

## How to use these in a repo.

In your WORKSPACE file, add:

```
http_archive(
   name = "libzmq",
   build_file = "@//:external/libzmq.BUILD",
   strip_prefix = "libzmq_prebuilt-0.1",
   urls = ["https://github.com/ngiyavuka/libzmq_prebuilt/archive/v0.1.tar.gz"],
   sha256 = "d3a6d17eaa96aad40426716c0d42b37971ca9f5575e20c6d254b0d25cd8afcb5"
)

# Example with cppzmq
http_archive(
   name = "zmq",
   build_file = "@//:external/zmq.BUILD",
   urls = ["https://github.com/zeromq/cppzmq/archive/v4.6.0.tar.gz"],
   strip_prefix = "cppzmq-4.6.0",
   patch_tool = "git apply",
   patches = ["@//:external/zmq.diff"],
   sha256 = "e9203391a0b913576153a2ad22a2dc1479b1ec325beb6c46a3237c669aef5a52",
)
```

For the example to work you'll need this patch file in your external directory to make bazel-compliant #include statements.
```
diff --git zmq.hpp zmq.hpp
index 74a0574..61e129c 100644
--- zmq.hpp
+++ zmq.hpp
@@ -69,7 +69,7 @@
 #define ZMQ_CONSTEXPR_VAR const
 #endif
 
-#include <zmq.h>
+#include "zmq.h"
 
 #include <cassert>
 #include <cstring>
```

Your `external/libzmq.BUILD`:
```
package(default_visibility = ['//visibility:public'])

cc_library(
    name = "libzmq",
    hdrs = ["zmq.h"],
    srcs = select({
        "@platforms//cpu:aarch64" : ["zeromq-4.3.3-aarch64-jetson-nano/libzmq.so"],
        "//conditions:default": ["zeromq-4.3.3-x86_64-Ubuntu-18.04.1/libzmq.so"],
    }),
)
```

and finally, for the example, your `external/zmq.BUILD`:
```
cc_library(
    name = "cppzmq",
    hdrs = [
        "zmq.hpp",
        "zmq_addon.hpp",
    ],
    visibility = ["//visibility:public"],
    deps = [
        "@libzmq//:libzmq",
    ],
    copts = [
        "-Iexternal/libzmq",
    ],
)
```


## To prebuild for more architectures

* Clone libzmq source to your target from https://github.com/zeromq/libzmq.
* Build with whatever commands are appropriate. In its simplest form, this will be a `./configure && make`
* Copy `src/.libs/libzmq.so` into a directory in this repo that describes the build.
