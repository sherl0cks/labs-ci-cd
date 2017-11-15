# Hoverfly

A simple fedora container to run [hoverfly](https://docs.hoverfly.io/en/latest/). 

TODO move to RHEL
TODO create a template

## Usage

### Running in OpenShift

Build the container and deploy it in openshift:

`$ oc new-app https://github.com/sherl0cks/labs-ci-cd#tool-box-zip --name=hoverfly --context-dir=docker/hoverfly`

Expose the proxy/webserver port

`$ oc expose svc hoverfly --port 8500`

Expose the admin interface, including the rest API

`$ oc expose svc hoverfly --port 8888 --name hoverfly-admin`

### Running in Docker

TODO

### Uploading Simulation Files

`$ curl -X PUT <hoverfly-proxy-webserver-route>/api/v2/simulation --upload-file <file-name>`

See [the docs](https://docs.hoverfly.io/en/latest/pages/reference/api/api.html) for detail

### Runtime Environment Variables

- `RUN_AS_WEBSERVER`: if set to any value, the hoverfly will start as a webserver. defaults to running as a proxy