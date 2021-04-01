# Goal

Build the image with a parameterized base image. 
That would give us 1 place in the tekton pipeline to set it for builds and scans.

The parameterized build only works in podman. I could not get it to use the parameter in openshift.

# Tekton Approach (works!)


Install Tekton Operator

    oc new-project haste-server
    oc adm policy add-scc-to-user privileged system:serviceaccount:haste-server:pipeline
    oc apply -f tekton-resource/pipeline-resources-pnst.yaml 
    oc apply -f tekton-resource/pipeline-resources-pnst.yaml 

https://github.com/containers/buildah/pull/2823

# BuildConfig based approach (does not work)

    spec:
      ..
      source:
        git:
          ref: build-arguments
          uri: https://github.com/kitty-catt/haste-server
        type: Git
      strategy:
        dockerStrategy:
          buildArgs:
          - name: BASE_IMAGE
            value: node:15.12.0-stretch
          env:
          - name: BUILD_LOGLEVEL
            value: "4"
          - name: BASE_IMAGE
            value: node:15.12.0-buster
          from:
            kind: ImageStreamTag
            name: node:14.8.0-stretch
        type: Docker


# Podman Procedure (works)

    git clone https://github.com/kitty-catt/haste-server
    git checkout build-arguments

    sudo podman build -t hastebin --build-arg  BASE_IMAGE=node:15.12.0-stretch .

    sudo podman run --name hastebin -p 8080:7777 localhost/hastebin
    open http://localhost:8080 in a browser

    sudo podman exec -it hastebin bash
    sudo podman stop hastebin
    sudo podman rm hastebin

    oc new-app --name=hastebin --as-deployment-config --build-env BASE_IMAGE=node:15.12.0-stretch https://github.com/kitty-catt/haste-server#build-arguments



# Appendix - 14.8.0-stretch

$ sudo podman inspect node:14.8.0-stretch
[
    {
        "Id": "784e696f50608aa3920d0fde0e2a9218409d166148559c34072ca71ea5577d17",
        "Digest": "sha256:8b6401f8d15c900736a54a870994277b3de19ebd28cc483c497bf00d608e2a90",
        "RepoTags": [
            "docker.io/library/node:14.8.0-stretch"
        ],
        "RepoDigests": [
            "docker.io/library/node@sha256:8b6401f8d15c900736a54a870994277b3de19ebd28cc483c497bf00d608e2a90",
            "docker.io/library/node@sha256:a3f4bcda6ea59aaf94ee8f9494ec12008557ef33647a43341a79002b6d7d0eed"
        ],

# Appendix - 15.12.0-stretch

$ sudo podman inspect node:15.12.0-stretch
[
    {
        "Id": "2530199d91c24f9d1843db28709e9d2d47b0ef59d9bb2bcdf208dee26474755a",
        "Digest": "sha256:35b326a83fc2d8d3f34b2b72c62e25eed19a3a83c501650edcd722242ed0249f",
        "RepoTags": [
            "docker.io/library/node:15.12.0-stretch"
        ],
        "RepoDigests": [
            "docker.io/library/node@sha256:35b326a83fc2d8d3f34b2b72c62e25eed19a3a83c501650edcd722242ed0249f",
            "docker.io/library/node@sha256:d66c11f62793ec5d00678a387b0279781b6b84190e720eb5422a4566c4bbc25f"
        ],
