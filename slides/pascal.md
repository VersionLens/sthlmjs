## Version Control for your full-stack JS app
How to version control a full-stack, multiple-service app, and run Versions with one-click
Note:
- Hi I'm Pascal and this is Fredrik, we work at a startup called Version Lens
- As you might tell from the name and this talk, Versions are very important to us
- I think all of us in this room think that version controlling source code is a no-brainer, our argument today is that version-controlling a full-stack app should be too.



What do we use for regular version control?
* Git <!-- .element: class="fragment" style="text-align: left;" -->

What do we use for version controlling <br/>multi-service apps? <!-- .element: class="fragment" -->
* GitOps <!-- .element: class="fragment" -->
Note:
- GitOps is a way of managing everything around the source code - build, deployment, infrastructure, etc, in a Git repository, and syncing that with the cloud.



What should a GitOps repository for a full-stack, <br/>multi-repository app contain?
1. Docker image build instructions for each service <!-- .element: class="fragment" -->
    - Note: Each service may be built from source code from one or more source repositories
1. Running instructions for the multi-service app as a whole <!-- .element: class="fragment" -->
    - Including instructions on how to repeatably "boot-up" the distributed app
Note:
- It comes down to Docker images - building and running them
- When we talk about a service, we mean a docker image and then a docker container - e.g. frontend, backend, database etc


Version Repository layout

![File layout of a Version Repository](/images/version-repo-layout.png "Version Repository Layout")
Note: 
- Context: let's imagine we're running a full-stack JS app, with a NodeJS backend and a SvelteKit frontend
- This is an e-commerce demo site, called The Version Store.
- We have two folders for the two steps: building images and running them
- This is an open source repository, you can use it as a template, see link at the end of this talk
- Let's look inside `build`


Version Repository `build/`
![File layout of build/](/images/version-repo-build.png "Version Repository build/")
Note: 
- Here we have two services: the frontend and the backend
- Let's look inside `version-store-backend`


Version Repository `build/{service}`
![File layout of build/{service}](/images/version-repo-build-backend.png "Version Repository build/{service}")
Note: 
- Two important files here: a workflow for building an image, and a file with parameters for that workflow.
- Let's look at the parameters first:


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
Note: 
- Source repositories on the bottom
- As you can see the commit hash is a variable
- Important: we use the commit hash from the source repository as the image tag. This is the core of GitOps, knowing exactly which version of a service is inside the Docker image, and then running as part of the app.


Building a service image
- We use Argo Workflows, which are like K8S Jobs on steroids <!-- .element: class="fragment" -->
- General steps are: <!-- .element: class="fragment" -->
  - Clone all source repositories
  - Docker build & push using Kaniko
- You could use any workflow system here, e.g. Tekton, etc. <!-- .element: class="fragment" -->
- We build all services in parallel, and steps can run in parallel if desired <!-- .element: class="fragment" -->
Note:
- K8s Jobs are like scripts that can run in a cluster
- K8s jobs don't scale very well in complexity, so we use Argo Workflow which is better
- Kaniko is Google's open-source in-cluster image builder, I believe they use this in Google Cloud Build


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
Note: Important here is that we check out a specific commit - again the core of GitOps.


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
Note:
- And that's it! We do the same for all the services, frontend and backend here, in parallel.
- Let's go back to the Version Repository and look in the other folder.



Version Repository layout

![File layout of a Version Repository](/images/version-repo-layout.png "Version Repository Layout")
Note: Here we click on `run`


Version Repository `run/`
![File layout of run/](/images/version-repo-run.png "Version Repository run/")
Note: 
- Important: building was separate, but now we're now dealing with all services at the same time
- Here we click on `params.jsonnet`


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
Note: 
- This is where we use the same commit SHA variables.


Running the multi-service app
- We use ArgoCD + Jsonnet <!-- .element: class="fragment" -->
- Why not helm / kustomize? <!-- .element: class="fragment" -->
    - They feel a bit like overkill to us but you could easily use them if you like
- You could also use something like Flux etc here instead <!-- .element: class="fragment" -->
Note:
- ArgoCD just looks at a folder with JSonnet or JSON files and keeps the manifests in sync with a k8s cluster.


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
Note: This is standard k8s, just note that we're using the image tag which is the commit SHA from the source repository.



## Summing up 

The point of a Version Repository is to make a "Version" of a multi-service app into a “function” of the git commits of its component services
