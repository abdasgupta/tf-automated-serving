# To build the image for tensorflow model server that can create containers to run tensorflow model server.
```bash
$ docker build -t tf-model-server -f Dockerfile .
```

# To collect the wheel file for inception model library so that inception-export.py can be run standalone from serving/tensorflow_serving/example
```bash
$ docker run -v /foo:/foo tf-model-server /bin/bash -c "cd /tf-automated-serving/inception && bazel-bin/inception/build-pip-package /foo"
```
