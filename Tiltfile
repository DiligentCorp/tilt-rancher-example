"""
An example Tiltfile (https://tilt.dev/)
"""

# Load the helm_remote extension so we can create services from remote Helm charts
load("ext://helm_remote", "helm_remote")

# Load the kim_build extension so we can build images using kim
load("ext://kim", "kim_build")

def standard_service(
        service_name,
        dockerfile = "Dockerfile",
        helm_values = [],
        live_update = False,
        container_path = "/usr/src/app",
        rebuild_on = [],
        **kwargs):
    """Build and deploy a service that follows a standard structure.

    Assumes the following:
      * Source code located at ./mern-stack-example/mern/<service_name>
      * Dockerfile located at ./pipeline/build/<service_name>/Dockerfile
      * Helm chart located at ./pipeline/helm-charts/<service_name>
      * Container image named registry.local/development/<service_name>

    Args:
      service_name: The name of the service to build and deploy. Used to
        construct the path to the source code directory
        (./mern-stack-example/mern/<service_name>, the path to the Helm chart
        (./pipeline/helm-charts/<service_name>),
        and the container image ref
        (registry.local/development/<service_name>).
      dockerfile: The name of the Dockerfile to use to build the service,
        located in ./pipeline/build/<service_name>. Defaults to 'Dockerfile'.
      helm_values: A list of strings, each of which is a path to a Helm
        values.yaml file to apply to the Helm chart, relative to the Tiltfile.
        Defaults to an empty list.
      live_update: A Boolean that indicates whether to perform live updates to
        running containers by copying in changed source files. Defaults to
        False.
      container_path: The full path inside the container at which to copy
        source files that have changed. Defaults to '/usr/src/app'. Only has an
        effect if live_update is True.
      rebuild_on: A list of strings, each of which is a path to a source code
        file, relative to the source code directory. Changing any of these
        files will trigger a full image rebuild. Only has an effect if
        live_update is True.
      **kwargs: Additional keyword arguments, passed unchanged to kim_build.

    Returns:
      None
    """

    image_name = "registry.local/development/{}".format(service_name)
    build_context = "./mern-stack-example/mern/{}".format(service_name)
    chart_path = "./pipeline/helm-charts/{}".format(service_name)
    dockerfile_path = "./pipeline/build/{}/{}".format(service_name, dockerfile)
    fall_back_paths = [
        "{}/{}".format(build_context, file)
        for file in rebuild_on
    ]

    build_args = dict(extra_flags = ["-f", dockerfile_path], **kwargs)

    if live_update:
        build_args["live_update"] = [
            fall_back_on(fall_back_paths),
            sync(build_context, container_path),
        ]

    k8s_yaml(helm(chart_path, values = helm_values))

    kim_build(image_name, build_context, **build_args)

# MongoDB from standard Helm chart
helm_remote(
    "mongodb",
    repo_name = "bitnami",
    repo_url = "https://charts.bitnami.com/bitnami",
    values = ["pipeline/values/mongodb.yaml"],
)

# MERN Example Server
standard_service(
    "server",
    helm_values = ["pipeline/values/common.yaml"],
    live_update = True,
    rebuild_on = ["package.json", "package-lock.lock"],
)
k8s_resource("server", port_forwards = "5000:5000")

# MERN Example Client
standard_service(
    "client",
    helm_values = [
        "pipeline/values/common.yaml",
        "pipeline/values/mern-example-client.yaml",
    ],
    live_update = True,
    rebuild_on = ["package.json", "package-lock.lock"],
)
k8s_resource("client", port_forwards = "3000:3000")
