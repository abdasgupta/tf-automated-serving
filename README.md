# To build the image for tensorflow model server binary that can create containers to run tensorflow model server.
```bash
$ docker build -t tf-model-server-bin -f Dockerfile_model_server_bin.ppc64le .
```

# To build and place the wheel file for inception model library in the host directory /foo
```bash
$ docker run -v /foo:/foo tf-model-server-bin /bin/bash -c "cd /tf-automated-serving/inception && bazel-bin/inception/build-pip-package /foo"
```
# To build and place the wheel file for inception client library in the host directory /foo
```bash
$ docker run -v /foo:/foo tf-model-server-bin /bin/bash -c "cd /tensorflow-serving && bazel-bin/tensorflow_serving/example/build-pip-package.runfiles/tf_serving/tensorflow_serving/example/build-pip-package /foo"
```
# To build and place the tensorflow model server binary in the host directory /foo
```bash
$ docker run -v /foo:/foo tf-model-server-bin /bin/bash -c "cp /tensorflow_model_server /foo"
```

# To build the image for inception exported checkpoint directory.
```bash
$ docker build -t tf-exported-ckpd-dir -f Dockerfile_inception_exported_dir.ppc64le .

```
# To build and place the wheel file for inception model library in the host directory /foo
```bash
$ docker run -v /foo:/foo tf-exported-ckpd-dir /bin/bash -c "cp -R /inception_export /foo"
```

# To build the image for combined inception checkpointed export directory and model server binary.
```bash
$ docker build -t tf-model-server -f Dockerfile_model_server_inception.ppc64le .

```
