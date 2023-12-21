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