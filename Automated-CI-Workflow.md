# Automated CI Workflow crib sheet

## Pull Requests Workflow

(Git Flow or GitHub Flow)
- Create a new branch from `main` for each new feature or bug fix.
- Commit changes to the new branch.
- Push the new branch to GitHub.
- Create a pull request from the new branch to `main`.
- Review the pull request. 
- Merge the pull request.

## Automation Events

- open pull request
- merge pull request

## Automation systems

- GitHub Actions
- Jenkins
- CircleCI
- Travis CI
- GitLab CI
- Bitbucket Pipelines
- AWS CodeBuild
- Azure Pipelines
- Google Cloud Build

## PR Workflows (Basic, Intermediate, Advanced)

### Basic

- open pull request
- run tests
- merge pull request

### Intermediate

- push to PR branch
    - lint 
    - image build 
    - unit tests
    - integration tests
    - security scan (cve scan)
- merge PR branch to main
    - lint 
    - image build / prod stage
    - push image to registry
    - deploy (1 or more environments)

### Advanced

- push to PR branch
    - lint 
    - image build 
       - unit tests
       - integration tests
       - security scan (cve scan)
       - deployment of smoke tests
    - SAST scan (security testing)
- merge PR branch to main
    - lint 
    - image build / prod stage
    - push image to registry
    - deploy (1 or more environments)

## GitHub Actions (GHA)

### Intro

- GitHub Actions is a CI/CD system built into GitHub.
- GitHub Actions is free for public repositories.

### Adding basic docker build

- Create a new file in the `.github/workflows` directory.
- Name the file `docker-build.yml`.
- Add the following content to the file:

```yaml
name: Docker Build

on:
  push:
    branches:
      - main
    pull_request:
      branches:
        - main

jobs:
    build-image:
        name: Build Docker Image
        runs-on: ubuntu-latest
        steps:
            - name: Login to DockerHub
              uses: docker/login-action@v1
              with:
                  username: ${{ secrets.DOCKERHUB_USERNAME }}
                  password: ${{ secrets.DOCKERHUB_TOKEN }}

            - name: Build Docker Image
                uses: docker/build-push-action@v2
                with:
                    push: ${{ github.event_name != 'pull_request' }}
                    tags: ${{ github.event_name != 'pull_request' && 'latest' }}
```

## GitHub Actions (GHA) - BuildKit Build Layer Caching

- BuildKit is a build engine for Docker.
- BuildKit can be used to cache layers during a build.

### Adding BuildKit Build Layer Caching (Speeds up builds)

- Create a new file in the `.github/workflows` directory.
- Name the file `docker-build.yml`.
- Add the following content to the file:

```yaml
name: Docker Build

on:
  push:
    branches:
      - main
    pull_request:
      branches:
        - main

jobs:
    build-image:
        name: Build Docker Image
        runs-on: ubuntu-latest
        steps:

            - name: Set up Docker BuildKit
              uses: docker/setup-buildx-action@v1

            - name: Login to DockerHub
              uses: docker/login-action@v1
              with:
                  username: ${{ secrets.DOCKERHUB_USERNAME }}
                  password: ${{ secrets.DOCKERHUB_TOKEN }}

            - name: Build Docker Image
                uses: docker/build-push-action@v2
                with:
                    push: ${{ github.event_name != 'pull_request' }}
                    tags: ${{ github.event_name != 'pull_request' && 'latest' }}
                    cache-from: type=gha
                    cache-to: type=gha,mode=max
```

- Commit and push the file to GitHub.
- These changes will add the following features:
    - BuildKit Build Layer Caching

### Adding Multi-Platform Build Support (ARM64, AMD64, etc.)

- Create a new file in the `.github/workflows` directory.
- Name the file `docker-build.yml`.
- Add the following content to the file:

```yaml
name: Docker Build

on:
  push:
    branches:
      - main
    pull_request:
      branches:
        - main

jobs:
    build-image:
        name: Build Docker Image
        runs-on: ubuntu-latest
        steps:

            - name: Set up QEMU
              uses: docker/setup-qemu-action@v1

            - name: Set up Docker BuildKit
              uses: docker/setup-buildx-action@v1

            - name: Login to DockerHub
              uses: docker/login-action@v1
              with:
                  username: ${{ secrets.DOCKERHUB_USERNAME }}
                  password: ${{ secrets.DOCKERHUB_TOKEN }}

            - name: Build Docker Image
                uses: docker/build-push-action@v2
                with:
                    push: ${{ github.event_name != 'pull_request' }}
                    tags: ${{ github.event_name != 'pull_request' && 'latest' }}
                    cache-from: type=gha
                    cache-to: type=gha,mode=max
                    platforms: linux/amd64,linux/arm64
```

- Note the addition of the line
- These changes will add the following features:
    - Multi-Platform Build Support

`platforms: linux/amd64,linux/arm64` - these are the same list of platform images that you see in Docker Hub.

- Commit and push the file to GitHub.

### Metadata and Dynamic Tags

- Create a new file in the `.github/workflows` directory.
- Name the file `docker-build.yml`.
- Add the following content to the file:

```yaml
name: Docker Build

on:
  push:
    branches:
      - main
    pull_request:
      branches:
        - main

jobs:
    build-image:
        name: Build Docker Image
        runs-on: ubuntu-latest
        steps:

            - name: Set up QEMU
              uses: docker/setup-qemu-action@v1

            - name: Set up Docker BuildKit
              uses: docker/setup-buildx-action@v1

            - name: Login to DockerHub
              uses: docker/login-action@v1
              with:
                  username: ${{ secrets.DOCKERHUB_USERNAME }}
                  password: ${{ secrets.DOCKERHUB_TOKEN }}

            - name: Docker meta
                id: meta
                uses: docker/metadata-action@v3
                with:
                    images: docker.io/${{ secrets.DOCKERHUB_USERNAME }}/hello-world
                    tags: |
                        type=raw,value=04
                        type=raw,value=latest, enable=${{ endsWith(github.ref, github.event.repository) }}
                        type=ref,event=pr
                        type=ref,event=branch
                        type=semver, prefix={{ version }}}
    
            - name: Build Docker Image
                uses: docker/build-push-action@v2
                with:
                    push: true
                    push: ${{ github.event_name != 'pull_request' }}
                    tags: ${{ github.event_name != 'pull_request' && 'latest' }}
                    cache-from: type=gha
                    cache-to: type=gha,mode=max
                    platforms: linux/amd64,linux/arm64
```

- Commit and push the file to GitHub.
- These changes will add the following features:
    - Metadata
    - Dynamic Tags

### Add GitHub PR Comments

### Metadata and Dynamic Tags

- Create a new file in the `.github/workflows` directory.
- Name the file `docker-build.yml`.
- Add the following content to the file:

```yaml
name: Docker Build

on:
  push:
    branches:
      - main
    pull_request:
      branches:
        - main

jobs:
    build-image:
        name: Build Docker Image
        runs-on: ubuntu-latest
        steps:

            - name: Set up QEMU
              uses: docker/setup-qemu-action@v1

            - name: Set up Docker BuildKit
              uses: docker/setup-buildx-action@v1

            - name: Login to DockerHub
              uses: docker/login-action@v1
              with:
                  username: ${{ secrets.DOCKERHUB_USERNAME }}
                  password: ${{ secrets.DOCKERHUB_TOKEN }}

            - name: Docker meta
                id: meta
                uses: docker/metadata-action@v3
                with:
                    images: docker.io/${{ secrets.DOCKERHUB_USERNAME }}/hello-world
                    tags: |
                        type=raw,value=04
                        type=raw,value=latest, enable=${{ endsWith(github.ref, github.event.repository) }}
                        type=ref,event=pr
                        type=ref,event=branch
                        type=semver, prefix={{ version }}}

            - name: Create or Update comment for image tags
                uses: peter-evans/create-or-update-comment@v1
                if: github.event_name == 'pull_request'
                with:
                    comment-id: ${{ steps.fc.outputs.comment_id }}
                    issue-number: ${{ github.event.pull_request.number }}
                    body: |
                        Docker image tag(s) pushed:
                        ```text
                        ${{ steps.docker_outputs.tags }}
                        ```
            
            - name: Build Docker Image
                uses: docker/build-push-action@v2
                with:
                    push: true
                    push: ${{ github.event_name != 'pull_request' }}
                    tags: ${{ github.event_name != 'pull_request' && 'latest' }}
                    cache-from: type=gha
                    cache-to: type=gha,mode=max
                    platforms: linux/amd64,linux/arm64
```

- Commit and push the file to GitHub.
- These changes will add the following features:
    - GitHub PR Comments

### Add CVE Scan

- Create a new file in the `.github/workflows` directory.
- Name the file `docker-build.yml`.
- Add the following content to the file:

```yaml
name: Docker Build

on:
  push:
    branches:
      - main
    pull_request:
      branches:
        - main

jobs:
    build-image:
        name: Build Docker Image
        runs-on: ubuntu-latest
        steps:

            - name: Set up QEMU
              uses: docker/setup-qemu-action@v1

            - name: Set up Docker BuildKit
              uses: docker/setup-buildx-action@v1

            - name: Login to DockerHub
              uses: docker/login-action@v1
              with:
                  username: ${{ secrets.DOCKERHUB_USERNAME }}
                  password: ${{ secrets.DOCKERHUB_TOKEN }}

            - name: Docker meta
                id: meta
                uses: docker/metadata-action@v3
                with:
                    images: docker.io/${{ secrets.DOCKERHUB_USERNAME }}/hello-world
                    tags: |
                        type=raw,value=04
                        type=raw,value=latest, enable=${{ endsWith(github.ref, github.event.repository) }}
                        type=ref,event=pr
                        type=ref,event=branch
                        type=semver, prefix={{ version }}}
            
            - name: Create or Update comment for image tags
                uses: peter-evans/create-or-update-comment@v1
                if: github.event_name == 'pull_request'
                with:
                    comment-id: ${{ steps.fc.outputs.comment_id }}
                    issue-number: ${{ github.event.pull_request.number }}
                    body: |
                        Docker image tag(s) pushed:
                        ```text
                        ${{ steps.docker_outputs.tags }}
                        ```
            
            - name: Run Trivy CVE Scan (none blocking)
                uses: aquasecurity/trivy-action@v0.3.0
                with:
                    image-ref: ${{ github.run_id }}
                    format: table
                    exit-code: 0
            
            - name: Build Docker Image
                uses: docker/build-push-action@v2
                with:
                    push: true
                    push: ${{ github.event_name != 'pull_request' }}
                    tags: ${{ github.event_name != 'pull_request' && 'latest' }}
                    cache-from: type=gha
                    cache-to: type=gha,mode=max
                    platforms: linux/amd64,linux/arm64
```

- Commit and push the file to GitHub.
- These changes will add the following features:
    - CVE Scan (Trivy) - non-blocking (exit-code: 0)

### Add CVE Scan (Blocking)

- Create a new file in the `.github/workflows` directory.
- Name the file `docker-build.yml`.
- Add the following content to the file:

```yaml
name: Docker Build

on:
  push:
    branches:
      - main
    pull_request:
      branches:
        - main

jobs:
    build-image:
        name: Build Docker Image
        runs-on: ubuntu-latest
        steps:

            permissions:
                contents: read # for actions/checkot to fetch code
                security-events: write # for github/codeql-action to upload SARIF results

            - name: Checkout git repo
              uses: actions/checkout@v2

            - name: Set up QEMU
              uses: docker/setup-qemu-action@v1

            - name: Set up Docker BuildKit
              uses: docker/setup-buildx-action@v1

            - name: Login to DockerHub
              uses: docker/login-action@v1
              with:
                  username: ${{ secrets.DOCKERHUB_USERNAME }}
                  password: ${{ secrets.DOCKERHUB_TOKEN }}

            - name: Docker meta
                id: meta
                uses: docker/metadata-action@v3
                with:
                    images: docker.io/${{ secrets.DOCKERHUB_USERNAME }}/hello-world
                    tags: |
                        type=raw,value=04
                        type=raw,value=latest, enable=${{ endsWith(github.ref, github.event.repository) }}
                        type=ref,event=pr
                        type=ref,event=branch
                        type=semver, prefix={{ version }}}
            
            - name: Create or Update comment for image tags
                uses: peter-evans/create-or-update-comment@v1
                if: github.event_name == 'pull_request'
                with:
                    comment-id: ${{ steps.fc.outputs.comment_id }}
                    issue-number: ${{ github.event.pull_request.number }}
                    body: |
                        Docker image tag(s) pushed:
                        ```text
                        ${{ steps.docker_outputs.tags }}
                        ```
            
            
            - name: Build Docker Image
                uses: docker/build-push-action@v2
                with:
                    push: false
                    load: true # Export to Docker Engine rather than pushing to registry
                    push: ${{ github.event_name != 'pull_request' }}
                    tags: ${{ github.event_name != 'pull_request' && 'latest' }}
                    cache-from: type=gha
                    cache-to: type=gha,mode=max
                    platforms: linux/amd64,linux/arm64

            - name: Run Trivy CVE Scan for CRITICAL and HIGH CVEs and report (blocking) 
                uses: aquasecurity/trivy-action@v0.3.0
                with:
                    image-ref: ${{ github.run_id }}
                    format: table
                    exit-code: 1
                    ignore-unfixed: true
                    vuln-type: os,library
                    severity: HIGH,CRITICAL
                    format: 'sarif'
                    output: 'trivy-results.sarif'

            - name: Upload Trivy SARIF results
                uses: github/codeql-action/upload-sarif@v1
                if: always() # Always upload results, even if previous step failed
                with:
                    sarif_file: trivy-results.sarif
```

- Commit and push the file to GitHub.
- These changes will add the following features:
    - CVE Scan (Trivy) - blocking (exit-code: 1)

### Add Unit Tests

- Create a new file in the `.github/workflows` directory.
- Name the file `docker-build.yml`.
- Add the following content to the file:

```yaml
name: Docker Build

on:
  push:
    branches:
      - main
    pull_request:
      branches:
        - main

jobs:
    build-image:
        name: Build Docker Image
        runs-on: ubuntu-latest
        steps:

            permissions:
                contents: read # for actions/checkot to fetch code
                security-events: write # for github/codeql-action to upload SARIF results

            - name: Checkout git repo
              uses: actions/checkout@v2

            - name: Set up QEMU
              uses: docker/setup-qemu-action@v1

            - name: Set up Docker BuildKit
              uses: docker/setup-buildx-action@v1

            - name: Login to DockerHub
              uses: docker/login-action@v1
              with:
                  username: ${{ secrets.DOCKERHUB_USERNAME }}
                  password: ${{ secrets.DOCKERHUB_TOKEN }}

            - name: Docker meta
                id: meta
                uses: docker/metadata-action@v3
                with:
                    images: docker.io/${{ secrets.DOCKERHUB_USERNAME }}/hello-world
                    tags: |
                        type=raw,value=04
                        type=raw,value=latest, enable=${{ endsWith(github.ref, github.event.repository) }}
                        type=ref,event=pr
                        type=ref,event=branch
                        type=semver, prefix={{ version }}}
            
            - name: Create or Update comment for image tags
                uses: peter-evans/create-or-update-comment@v1
                if: github.event_name == 'pull_request'
                with:
                    comment-id: ${{ steps.fc.outputs.comment_id }}
                    issue-number: ${{ github.event.pull_request.number }}
                    body: |
                        Docker image tag(s) pushed:
                        ```text
                        ${{ steps.docker_outputs.tags }}
                        ```
            
            
            - name: Build Docker Image
                uses: docker/build-push-action@v2
                with:
                    push: false
                    load: true # Export to Docker Engine rather than pushing to registry
                    push: ${{ github.event_name != 'pull_request' }}
                    tags: ${{ github.event_name != 'pull_request' && 'latest' }}
                    cache-from: type=gha
                    cache-to: type=gha,mode=max
                    platforms: linux/amd64,linux/arm64

            - name: Unit Testing in Docker
                run: |
                    docker run --rm ${{ github.run_id }} echo "run test commands here"
                
                        - name: Build Docker Image
                uses: docker/build-push-action@v2
                with:
                    push: false
                    load: true # Export to Docker Engine rather than pushing to registry
                    push: ${{ github.event_name != 'pull_request' }}
                    tags: ${{ github.event_name != 'pull_request' && 'latest' }}
                    cache-from: type=gha
                    cache-to: type=gha,mode=max
                    platforms: linux/amd64,linux/arm64

            - name: Run Trivy CVE Scan for CRITICAL and HIGH CVEs and report (blocking) 
                uses: aquasecurity/trivy-action@v0.3.0
                with:
                    image-ref: ${{ github.run_id }}
                    format: table
                    exit-code: 1
                    ignore-unfixed: true
                    vuln-type: os,library
                    severity: HIGH,CRITICAL
                    format: 'sarif'
                    output: 'trivy-results.sarif'

            - name: Upload Trivy SARIF results
                uses: github/codeql-action/upload-sarif@v1
                if: always() # Always upload results, even if previous step failed
                with:
                    sarif_file: trivy-results.sarif

            - name: Docker Metadata for Final Image Build
                id: docker_meta
                uses: docker/build-push-action@v3
                with:
                    image: ${{ secrets.DOCKERHUB_USERNAME }}/hello-world
                    flavor: |
                        latest=false
                    tags:
                        type=raw,value=08
```

- Commit and push the file to GitHub.
- These changes will add the following features:
    - Unit Tests

### Add Integration Tests

- Create a new file in the `.github/workflows` directory.
- Name the file `docker-build.yml`.
- Add the following content to the file:

```yaml
name: Docker Build

on:
  push:
    branches:
      - main
    pull_request:
      branches:
        - main

jobs:
    build-image:
        name: Build Docker Image
        runs-on: ubuntu-latest
        steps:

            permissions:
                contents: read # for actions/checkot to fetch code
                security-events: write # for github/codeql-action to upload SARIF results

            - name: Checkout git repo
              uses: actions/checkout@v2

            - name: Set up QEMU
              uses: docker/setup-qemu-action@v1

            - name: Set up Docker BuildKit
              uses: docker/setup-buildx-action@v1

            - name: Login to DockerHub
              uses: docker/login-action@v1
              with:
                  username: ${{ secrets.DOCKERHUB_USERNAME }}
                  password: ${{ secrets.DOCKERHUB_TOKEN }}

            - name: Docker meta
                id: meta
                uses: docker/metadata-action@v3
                with:
                    images: docker.io/${{ secrets.DOCKERHUB_USERNAME }}/hello-world
                    tags: |
                        type=raw,value=04
                        type=raw,value=latest, enable=${{ endsWith(github.ref, github.event.repository) }}
                        type=ref,event=pr
                        type=ref,event=branch
                        type=semver, prefix={{ version }}}
            
            - name: Create or Update comment for image tags
                uses: peter-evans/create-or-update-comment@v1
                if: github.event_name == 'pull_request'
                with:
                    comment-id: ${{ steps.fc.outputs.comment_id }}
                    issue-number: ${{ github.event.pull_request.number }}
                    body: |
                        Docker image tag(s) pushed:
                        ```text
                        ${{ steps.docker_outputs.tags }}
                        ```
            
            
            - name: Build Docker Image
                uses: docker/build-push-action@v2
                with:
                    push: false
                    load: true # Export to Docker Engine rather than pushing to registry
                    push: ${{ github.event_name != 'pull_request' }}
                    tags: ${{ github.event_name != 'pull_request' && 'latest' }}
                    cache-from: type=gha
                    cache-to: type=gha,mode=max
                    platforms: linux/amd64,linux/arm64

            - name: Unit Testing in Docker
                run: |
                    docker run --rm ${{ github.run_id }} echo "run test commands here"
                
                        - name: Build Docker Image
                uses: docker/build-push-action@v2
                with:
                    push: false
                    load: true # Export to Docker Engine rather than pushing to registry
                    push: ${{ github.event_name != 'pull_request' }}
                    tags: ${{ github.event_name != 'pull_request' && 'latest' }}
                    cache-from: type=gha
                    cache-to: type=gha,mode=max
                    platforms: linux/amd64,linux/arm64

            - name: Test healthcheck in Docker Compose
                run: |
                    export TESTING_IMAGE=$DOCKERHUB_USERNAME/hello-world:${{ steps.docker_meta.outputs.tags }}
                    docker-compose -f docker-compose.test.yml up --abort-on-container-exit --exit-code-from sut

            - name: Run Trivy CVE Scan for CRITICAL and HIGH CVEs and report (blocking) 
                uses: aquasecurity/trivy-action@v0.3.0
                with:
                    image-ref: ${{ github.run_id }}
                    format: table
                    exit-code: 1
                    ignore-unfixed: true
                    vuln-type: os,library
                    severity: HIGH,CRITICAL
                    format: 'sarif'
                    output: 'trivy-results.sarif'

            - name: Upload Trivy SARIF results
                uses: github/codeql-action/upload-sarif@v1
                if: always() # Always upload results, even if previous step failed
                with:
                    sarif_file: trivy-results.sarif

            - name: Docker Metadata for Final Image Build
                id: docker_meta
                uses: docker/build-push-action@v3
                with:
                    image: ${{ secrets.DOCKERHUB_USERNAME }}/hello-world
                    flavor: |
                        latest=false
                    tags:
                        type=raw,value=08
```

- Commit and push the file to GitHub.
- These changes will add the following features:
    - Integration Tests
- Corrosponding files required:
    - `docker-compose.test.yml`

### Add Kubenetes Smoke Tests

- You could use two different registries for two different purposes:
    - `docker.io` for production images (already there for Docker Hub)
    - `ghcr.io` for smoke test images (already there for GitHub Actions)

- Create a new file in the `.github/workflows` directory.
- Name the file `docker-build.yml`.
- Add the following content to the file:

```yaml
name: Docker Build

on:
  push:
    branches:
      - main
    pull_request:
      branches:
        - main

jobs:
    build-image:
        name: Build Docker Image
        runs-on: ubuntu-latest
        steps:

            permissions:
                contents: read # for actions/checkot to fetch code
                security-events: write # for github/codeql-action to upload SARIF results

            - name: Checkout git repo
              uses: actions/checkout@v2

            - name: Set up QEMU
              uses: docker/setup-qemu-action@v1

            - name: Set up Docker BuildKit
              uses: docker/setup-buildx-action@v1

            - name: Login to DockerHub
              uses: docker/login-action@v1
              with:
                  username: ${{ secrets.DOCKERHUB_USERNAME }}
                  password: ${{ secrets.DOCKERHUB_TOKEN }}

            - name: Docker meta
                id: meta
                uses: docker/metadata-action@v3
                with:
                    images: docker.io/${{ secrets.DOCKERHUB_USERNAME }}/hello-world
                    tags: |
                        type=raw,value=04
                        type=raw,value=latest, enable=${{ endsWith(github.ref, github.event.repository) }}
                        type=ref,event=pr
                        type=ref,event=branch
                        type=semver, prefix={{ version }}}
            
            - name: Create or Update comment for image tags
                uses: peter-evans/create-or-update-comment@v1
                if: github.event_name == 'pull_request'
                with:
                    comment-id: ${{ steps.fc.outputs.comment_id }}
                    issue-number: ${{ github.event.pull_request.number }}
                    body: |
                        Docker image tag(s) pushed:
                        ```text
                        ${{ steps.docker_outputs.tags }}
                        ```
            
            
            - name: Build Docker Image
                uses: docker/build-push-action@v2
                with:
                    push: false
                    load: true # Export to Docker Engine rather than pushing to registry
                    push: ${{ github.event_name != 'pull_request' }}
                    tags: ${{ github.event_name != 'pull_request' && 'latest' }}
                    cache-from: type=gha
                    cache-to: type=gha,mode=max
                    platforms: linux/amd64,linux/arm64

            - name: Unit Testing in Docker
                run: |
                    docker run --rm ${{ github.run_id }} echo "run test commands here"
                
                        - name: Build Docker Image
                uses: docker/build-push-action@v2
                with:
                    push: false
                    load: true # Export to Docker Engine rather than pushing to registry
                    push: ${{ github.event_name != 'pull_request' }}
                    tags: ${{ github.event_name != 'pull_request' && 'latest' }}
                    cache-from: type=gha
                    cache-to: type=gha,mode=max
                    platforms: linux/amd64,linux/arm64

            - name: Test healthcheck in Docker Compose
                run: |
                    export TESTING_IMAGE=$DOCKERHUB_USERNAME/hello-world:${{ steps.docker_meta.outputs.tags }}
                    docker-compose -f docker-compose.test.yml up --abort-on-container-exit --exit-code-from sut

            - name: Run Trivy CVE Scan for CRITICAL and HIGH CVEs and report (blocking) 
                uses: aquasecurity/trivy-action@v0.3.0
                with:
                    image-ref: ${{ github.run_id }}
                    format: table
                    exit-code: 1
                    ignore-unfixed: true
                    vuln-type: os,library
                    severity: HIGH,CRITICAL
                    format: 'sarif'
                    output: 'trivy-results.sarif'

            - name: Upload Trivy SARIF results
                uses: github/codeql-action/upload-sarif@v1
                if: always() # Always upload results, even if previous step failed
                with:
                    sarif_file: trivy-results.sarif

            - name: Docker Metadata for Final Image Build
                id: docker_meta
                uses: docker/build-push-action@v3
                with:
                    image: ${{ secrets.DOCKERHUB_USERNAME }}/hello-world
                    flavor: |
                        latest=false
                    tags:
                        type=raw,value=08

            - name: Docker Metadata for Smoke Test Image
                id: docker_meta_smoke
                uses: docker/build-push-action@v3
                with:
                    image: ghcr.io/${{ secrets.DOCKERHUB_USERNAME }}/hello-world
                    flavor: |
                        latest=false
                    tags:
                        type=raw,value=08
```

- Commit and push the file to GitHub.
- These changes will add the following features:
    - Kubernetes Smoke Tests

### Parallelise Jobs

- You can run jobs in parallel by adding the `needs` keyword to the job.  
- The `needs` keyword is a list of jobs that must complete before the current job can start.
- The `needs` keyword can be used to run jobs in parallel.
- The `needs` keyword can be used to run jobs in sequence.
- The `needs` keyword can be used to run jobs in a combination of parallel and sequence.

- Create a new file in the `.github/workflows` directory.
- Name the file `docker-build.yml`.
- Add the following content to the file:

```yaml
---
name: 99 Parallelize Jobs

on:
  push:
    branches:
      - main
  pull_request:

jobs:
# FIRST JOB #######################################################################   
  build-test-image:
    name: Build Image for Testing
    runs-on: ubuntu-latest

    permissions:
      packages: write # needed to push docker image to ghcr.io

    steps:

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Login to ghcr.io registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Build and Push to GHCR
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: ghcr.io/helloworld:${{ github.run_id }}
          target: test
          cache-from: type=gha
          cache-to: type=gha,mode=max
          platforms: linux/amd64
    
 # NEXT JOB #######################################################################   
  test-unit:
    name: Unit tests in Docker
    needs: [build-test-image]
    runs-on: ubuntu-latest

    permissions:
      packages: read
      
    steps:
      
      - name: Login to ghcr.io registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Unit Testing in Docker
        run: docker run --rm ghcr.io/helloworld:"$GITHUB_RUN_ID" echo "run test commands here"

# NEXT JOB #######################################################################   
  test-integration:
    name: Integration tests in Compose
    needs: [build-test-image]
    runs-on: ubuntu-latest

    permissions:
      packages: read

    steps:

      - name: Checkout git repo
        uses: actions/checkout@v4

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      
      - name: Login to ghcr.io registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Test healthcheck in Docker Compose
        run: |
          export TESTING_IMAGE=ghcr.iohelloworld:"$GITHUB_RUN_ID"
          echo Testing image: "$TESTING_IMAGE"
          docker compose -f docker-compose.test.yml up --exit-code-from sut

# NEXT JOB #######################################################################   
  test-k3d:
    name: Test Deployment in Kubernetes
    needs: [build-test-image]
    runs-on: ubuntu-latest

    permissions:
      packages: read

    steps:

      - name: Checkout git repo
        uses: actions/checkout@v4

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      
      - name: Login to ghcr.io registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - uses: AbsaOSS/k3d-action@v2
        with:
          cluster-name: "test-cluster-1"
          args: >-
            --agents 1
            --no-lb
            --k3s-arg "--no-deploy=traefik,servicelb,metrics-server@server:*"
      
      - name: Smoke test deployment in k3d Kubernetes
        run: |
          kubectl create secret docker-registry regcred \
            --docker-server=https://ghcr.io \
            --docker-username=${{ github.actor }} \
            --docker-password=${{ secrets.GITHUB_TOKEN }}
          export TESTING_IMAGE=ghcr.iohelloworld:"$GITHUB_RUN_ID"
          envsubst < manifests/deployment.yaml  | kubectl apply -f -
          kubectl rollout status deployment myapp
          kubectl exec deploy/myapp -- curl --fail localhost

# NEXT JOB #######################################################################   
  scan-image:
    name: Scan Image with Trivy
    needs: [build-test-image]
    runs-on: ubuntu-latest

    permissions:
      contents: read # for actions/checkout to fetch code
      packages: read # needed to pull docker image to ghcr.io
      security-events: write # for github/codeql-action/upload-sarif to upload SARIF results

    steps:

      - name: Checkout git repo
        uses: actions/checkout@v4
      
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
          
      - name: Login to ghcr.io registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Pull image to scan
        run: docker pull ghcr.iohelloworld:"$GITHUB_RUN_ID"
        
      - name: Run Trivy for all CVEs (non-blocking)
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ghcr.iohelloworld:${{ github.run_id }}
          format: table
          exit-code: 0

      # NOTE: disabled to avoid running twice and overwriting 07-add-cve-scanning-adv.yaml
      # - name: Run Trivy for HIGH,CRITICAL CVEs and report (blocking)
      #   uses: aquasecurity/trivy-action@master
      #   with:
      #     image-ref: ghcr.iohelloworld:${{ github.run_id }}
      #     exit-code: 1
      #     ignore-unfixed: true
      #     vuln-type: 'os,library'
      #     severity: 'HIGH,CRITICAL'
      #     format: 'sarif'
      #     output: 'trivy-results.sarif'
      
      # - name: Upload Trivy scan results to GitHub Security tab
      #   uses: github/codeql-action/upload-sarif@v1
      #   if: always()
      #   with:
      #     sarif_file: 'trivy-results.sarif'

# NEXT JOB #######################################################################   
  build-final-image:
    name: Build Final Image
    needs: [test-unit, test-integration, test-k3d, scan-image]
    runs-on: ubuntu-latest

    permissions:
      packages: write # needed to push docker image to ghcr.io
      pull-requests: write # needed to create and update comments in PRs

    steps:

      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Login to ghcr.io registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Docker Metadata for Final Image Build
        id: docker_meta
        uses: docker/metadata-action@v5
        with:
          images: ghcr.io/hello-world
          flavor: |
            latest=false
          tags: |
            type=raw,value=99
          # comment these out on all but 04-add-metadata.yaml to avoid overwriting image tags
          # type=raw,value=latest,enable=${{ endsWith(github.ref, github.event.repository.default_branch) }}
          # type=ref,event=pr
          # type=ref,event=branch
          # type=semver,pattern={{version}}
      
      - name: Docker Build and Push to GHCR and Docker Hub
        uses: docker/build-push-action@v5
        with:
          push: true
          tags: ${{ steps.docker_meta.outputs.tags }}
          labels: ${{ steps.docker_meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
          platforms: linux/amd64,linux/arm64,linux/arm/v7

        # If PR, put image tags in the PR comments
        # from https://github.com/marketplace/actions/create-or-update-comment
        # These are commented out to avoid conflicting with 05-add-comments.yaml example
      # - name: Find comment for image tags
      #   uses: peter-evans/find-comment@v1
      #   if: github.event_name == 'pull_request'
      #   id: fc
      #   with:
      #     issue-number: ${{ github.event.pull_request.number }}
      #     comment-author: 'github-actions[bot]'
      #     body-includes: Docker image tag(s) pushed
      
      #   # If PR, put image tags in the PR comments
      # - name: Create or update comment for image tags
      #   uses: peter-evans/create-or-update-comment@v1
      #   if: github.event_name == 'pull_request'
      #   with:
      #     comment-id: ${{ steps.fc.outputs.comment-id }}
      #     issue-number: ${{ github.event.pull_request.number }}
      #     body: |
      #       Docker image tag(s) pushed:
      #       ```text
      #       ${{ steps.docker_meta.outputs.tags }}
      #       ```

      #       Labels added to images:
      #       ```text
      #       ${{ steps.docker_meta.outputs.labels }}
      #       ```
      #     edit-mode: replace
```

### Docker Security Good Defaults and Tools