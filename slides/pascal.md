## "Version Repository"
### A GitOps repository for a full-stack, multi-repository app



What should a GitOps repository for a full-stack, <br/>multi-repository app contain?
1. Docker image build instructions for each service <!-- .element: class="fragment" -->
    - Note: Each service may be built from source code from one or more source repositories
1. Running instructions for the multi-service app as a whole <!-- .element: class="fragment" -->
    - Including instructions on how to repeatably "boot-up" the distributed app



Version Repository layout

![File layout of a Version Repository](/images/version-repo-layout.png "Version Repository Layout")
Note: Here we click on `build`



Version Repository `build/`
![File layout of build/](/images/version-repo-build.png "Version Repository build/")
Note: Here we click on `version-store-backend`


Version Repository `build/{service}`
![File layout of build/{service}](/images/version-repo-build-backend.png "Version Repository build/{service}")
Note: Here we click on `build_params.yaml`


Version Repository `build/{service}/build_params.yaml`
<pre><code data-trim data-noescape>
image_params:
  version_store_backend:
    docker_build_context: ""
    dockerfile_path: Dockerfile
    image: versionlens/version-store-backend-nodejs
    registry: index.docker.io
repo_dependencies:
  version_store_backend:
    commit: $VERSION_STORE_BACKEND_SHA
    url: https://github.com/VersionLens/version-store-backend-nodejs.git
</code></pre>


Building a service image
- We use Argo Workflows, which are like K8S Jobs on steroids <!-- .element: class="fragment" -->
- General steps are: <!-- .element: class="fragment" -->
  - Clone all source repositories
  - Docker build & push using Kaniko
- You could use any workflow system here, e.g. Tekton, etc. <!-- .element: class="fragment" -->
- We build all services in parallel, and steps can run in parallel if desired <!-- .element: class="fragment" -->


Version Repository `build/{service}/build.argo.yaml`

<pre><code data-trim data-noescape>
- name: clone-github-repo
  ...
  script:
    image: alpine/git
    command: [ sh ]
    source: |
      git clone -n --progress {{repo}} /workspace
      cd /workspace
      git checkout {{commit}}
    volumeMounts:
    - name: workspace
        mountPath: /workspace
</code></pre>


Version Repository `build/{service}/build.argo.yaml`

<pre><code data-trim data-noescape>
- name: build-push-docker
  ...
  container:
    image: gcr.io/kaniko-project/executor:latest
    command: [/kaniko/executor]
    args:
      //...dockerfile, destination, context
    workingDir: /workspace
    volumeMounts:
      - name: workspace
        mountPath: /workspace
      - name: versionlens-regcred
        mountPath: /kaniko/.docker/
</code></pre>



Version Repository layout

![File layout of a Version Repository](/images/version-repo-layout.png "Version Repository Layout")
Note: Here we click on `run`



Version Repository `run/`
![File layout of run/](/images/version-repo-run.png "Version Repository run/")
Note: Here we click on `params.jsonnet`


Version Repository `run/params.jsonnet`
<pre><code data-trim data-noescape>
{
  version_store_backend: {
    image: 'versionlens/version-store-backend-nodejs',
    name: 'version-store-backend',
    registry: 'docker.io',
    tag: std.extVar('VERSION_STORE_BACKEND_SHA'),
  },
  version_store_frontend: {
    image: 'versionlens/version-store-frontend',
    name: 'version-store-frontend',
    registry: 'docker.io',
    tag: std.extVar('VERSION_STORE_FRONTEND_SHA'),
  },
}
</code></pre>


Running the multi-service app
- We use ArgoCD + Jsonnet <!-- .element: class="fragment" -->
- Why not helm / kustomize? <!-- .element: class="fragment" -->
    - They feel a bit like overkill but you could easily use them if you like
- You could also use something like Flux etc here instead <!-- .element: class="fragment" -->


How do we orchestrate the boot-up<br/> of a multi-service app?
- Standard k8s: <!-- .element: class="fragment" -->
    - Init containers, e.g. wait for a database
    - k8s Jobs, e.g. to seed a database
- You could easily run integration/e2e tests as a Job <!-- .element: class="fragment" -->


Version Repository `run/{service}-deployment.jsonnet`

<pre><code data-trim data-noescape>
local params = import 'params.jsonnet';

{
  apiVersion: 'apps/v1',
  kind: 'Deployment',
    containers: [
      {
        name: params.version_store_frontend.name,
        image: params.version_store_frontend.registry + '/' 
             + params.version_store_frontend.image 
             + ':' + params.version_store_frontend.tag,
        env: ...,
      },
    ],
  ...
</code></pre>



## Summing up 

The point of a Version Repository is to make a "Version" of a multi-service app into a “function” of the git commits of its component services



Version Repository `params.yaml`

<pre><code data-trim data-noescape>
version-store-frontend:
  - name: VERSION_STORE_FRONTEND_SHA
    type: GIT_COMMIT_SHA
    description: The commit hash of the version-store-frontend repo, e.g. ff58cc556269ede04ef6045465aafb2ed5747903
version-store-backend:
  - name: VERSION_STORE_BACKEND_SHA
    type: GIT_COMMIT_SHA
    description: The commit hash of the version-store-backend repo, e.g. 142b56b046d32eef4dc159359cb29d54a4c02b98
</code></pre>