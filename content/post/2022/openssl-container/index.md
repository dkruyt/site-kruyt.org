---
title: "OpenSSL vulnerability don't forget your containers"
date: 2022-11-02T09:15:15+02:00
draft: false
author: Dennis Kruyt
categories:
- containers
cover:
  image: /images/2022/11/container-fail.jpg
description: ""
slug: numpads
summary: OpenSSL vulnerability don't forget your containers
tags:
- containers
- openssl
- security
---

Yesterday OpenSSL released version 3.0.7 that fixes vulnerability CVE-2022-37786 and CVE-2022-3602. For all major Linux distro where updates available. So a dnf/apt update/upgrade fixes those issue's. So now you are safe? Well maybe, do you run also containers in docker/kubernetes? Those containers are also running a kind of mini OS.

So maybe your contaier host is patched, but the containers it self not. 


https://github.com/NCSC-NL/OpenSSL-2022/blob/main/software/README.md

Trivy is a CLI tool that can be used to scan your container images or running containers for vulnerabilities and outdated packages. In this blog post, we will show you how to use Trivy to scan your Kubernetes containers for vulnerabilities and outdated packages.

First, you will need to install Trivy on your Kubernetes cluster. To do this, you can use the kubectl command-line tool to create a Trivy deployment:
```
# create the Trivy deployment
$ kubectl apply -f https://raw.githubusercontent.com/aquasecurity/trivy-helm/master/contrib/trivy/templates/deployment.yaml

# check the status of the deployment
$ kubectl get deployments -l app=trivy
```
This will create a Trivy deployment on your Kubernetes cluster, which you can use to scan your containers.

Next, you can use Trivy to scan your containers for vulnerabilities and outdated packages. To do this, you can create a Trivy job, which will run Trivy inside a container and scan the specified containers. Here is an example of how to create a Trivy job to scan the containers in a deployment:
```
# create a Trivy job
$ cat <<EOF | kubectl apply -f -
apiVersion: batch/v1
kind: Job
metadata:
  name: trivy-scan
spec:
  template:
    metadata:
      name: trivy-scan
    spec:
      containers:
      - name: trivy-scan
        image: aquasec/trivy
        args: ["--quiet", "--exit-code=1", "--severity=HIGH", "--format=json", "--ignore-unfixed", "--skip-update", "--no-progress", "--file-output=trivy-results.json", "--target=deployment/my-app"]
      restartPolicy: Never
EOF
```

This Trivy job will run the trivy command inside a container and scan the containers in the my-app deployment for vulnerabilities and outdated packages. The --quiet, --exit-code=1, and --severity=HIGH flags will make Trivy only output the vulnerabilities with a high severity, and the --format=json and --file-output=trivy-results.json flags will make Trivy output the scan results in JSON format to the trivy-results.json file.

Once the Trivy job is complete, you can view the scan results by getting the logs of the Trivy job:
```
# check the status of the job
$ kubectl get jobs -l app=trivy

# get the logs of the job
$ kubectl logs -l app=trivy
```

The logs of the Trivy job will contain the JSON output of the scan results, which you can then parse and analyze to see which vulnerabilities and outdated packages were found in your containers.

Overall, using Trivy to scan your Kubernetes containers for vulnerabilities and outdated packages is a quick and easy way to ensure that your containers are secure and up-to-date. By regularly running Trivy scans, you can keep your containers free of vulnerabilities and ensure that they are always running the latest, most secure versions of their dependencies.


you can use Trivy in the container build stage of a GitLab CI/CD pipeline. To do this, you can add a Trivy step to your pipeline that runs Trivy against your container image before it is pushed to a registry.

Here is an example of a GitLab pipeline that uses Trivy to scan a container image during the build stage:

```
stages:
  - build
  - test
  - deploy

build:
  stage: build
  script:
    # build the container image
    - docker build -t my-app .

    # scan the container image with Trivy
    - trivy --exit-code=1 --severity=HIGH my-app
  artifacts:
    paths:
      - my-app

test:
  stage: test
  script:
    # run tests against the container image
    - docker run my-app sh -c "run tests"
  dependencies:
    - build

deploy:
  stage: deploy
  script:
    # push the container image to a registry
    - docker push my-app
  dependencies:
    - test
```

In this pipeline, the build stage builds the my-app container image and then uses Trivy to scan the image for vulnerabilities and outdated packages. If Trivy finds any vulnerabilities with a high severity, the pipeline will be failed. Otherwise, the pipeline will continue to the test stage, where the container image is tested, and then to the deploy stage, where the image is pushed to a registry.

This is just one example of how you can use Trivy in a GitLab pipeline to scan your container images for vulnerabilities and outdated packages. You can customize and extend this pipeline as needed to fit your specific CI/CD workflow. For example, you could add additional steps to notify team members of the scan results, or to automatically update the container image if any outdated packages were found.

