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

## GitHub Actions

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

    
- Commit and push the file to GitHub.