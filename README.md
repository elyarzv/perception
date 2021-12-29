# Phoenix-perception-binder
# Development Workflow
- Run these commands inside the phoenix binder folder:
```
make src-update-submodules
export TAG=latest
make docker-start-dev
make docker-enter-dev
make build
source devel/setup.bash
```
- Stop the Docker container when you are done:
```
make docker-stop-dev
```

# Building and Pushing Docker Images

```
make src-update-submodules
make docker-build-base
make docker-push-base
make docker-build-dev
make docker-push-dev
make docker-build-prod
make docker-push-prod
```

# Running Docker Containers

- Running the base Docker image:
```
   make docker-start-base
```
- Running the development Docker image:
```
   make docker-start-dev
```
- Running the production Docker image:
```
   make docker-start-prod
```

# Opening a terminal on Docker Containers

- Entering the base Docker container:
```
   make docker-enter-base
```
- Entering the development Docker container:
```
   make docker-enter-dev
```
- Entering the production Docker container:
```
   make docker-enter-prod
```

# Stopping Docker Containers

- Stopping the base Docker container:
```
   make docker-stop-base
```
- Stopping the development Docker container:
```
   make docker-stop-dev
```
- Stopping the production Docker container:
```
   make docker-stop-prod
```

# Developing inside the container as $USER for the robot
- Build the whole workspace
```
export TAG=develop* --> Please take a look at the section after developing for the simulation
export ROBOT_NAME=condor
make docker-pull-dev --> Skip this step if you want to use your local image
make make docker-start-robot-cpu --> Use TAG=local if you built the image locally
make docker-enter-dev
make build-robot-workspace
source robot_ws/install/setup.bash
make build-simulation-workspace
source simulation_ws/install/setup.bash
export SIMULATION=false
export DEVELOPING=true
roslaunch phoenix1_bringup bringup.launch
```

# Developing inside the container as $USER for the simulation
- Build the whole workspace
```
export TAG=develop* --> Please take a look at the next section
export ROBOT_NAME=condor
make docker-pull-dev --> Skip this step if you want to use your local image
make docker-start-dev --> Use TAG=local if you built the image locally
make docker-enter-dev
make build-robot-workspace
source robot_ws/install/setup.bash
make build-simulation-workspace
source simulation_ws/install/setup.bash
export SIMULATION=true
export DEVELOPING=true
roslaunch phoenix1_bringup bringup.launch

```
-  Attention developers
```export TAG=develop``` will pull ```62427299064.dkr.ecr.ca-central-1.amazonaws.com/binders:cartpuller-dev-develop``` from AWS. If you used Jira to create a branch or the branch starts with feature, hotfix, bugfix, or release it will automatically build on the Jenkins pipeline. You can pull this image using: ```export TAG=feature-OCP-100-your-branch``` for the branch: ```feature/OCP-100-your-branch``` and then ```make docker-pull-dev```
For this to work the jenkins build should have finished, otherwise you can `export TAG=develop` or `export TAG=master`
[This is where it is used in Jenkins](https://bitbucket.org/ais_admin/cartpuller-binder/src/2591da86a867c2618c2454da26b7cc4eee9bd9a1/binder/jenkins/Jenkinsfile#lines-25) and [in the ais-makefile](https://bitbucket.org/ais_admin/ais-makefile/src/becabc1dd7adb917727dcb543dfd9e9a70a7f212/Makefile#lines-165)

# Infrastructure
The jenkinks server can be found in the following address `http://build.ais-api.com:8080/`

# Note to ADE
As of `2021-09-01` [ADE is compatible with Gitlab and Dockerhub registries](https://gitlab.com/ApexAI/ade-cli/-/blob/master/ade_cli/credentials.py#L90-100), to overcome this build the dev pipeline locally with
```
make docker-login # This command will log you inside the AWS ECR registry
make src-update-submodules
make docker-build-dev # Builds the development image
```
Or download it from the docker registry and tag it again
```
docker pull 062427299064.dkr.ecr.ca-central-1.amazonaws.com/binders:phoenix1-binder-dev
docker tag 062427299064.dkr.ecr.ca-central-1.amazonaws.com/binders:phoenix1-binder-dev binders:phoenix1-binder-dev
```

Then use [ade](https://ade-cli.readthedocs.io/en/latest/#) as usual.  
