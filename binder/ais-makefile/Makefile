.DEFAULT_GOAL := help
SHELL=/bin/bash

TAG?=local
DOCKERHUB_REGISTRY=aisadmin/${IMAGE_REPO_NAME}
AWS_ACCOUNT_ID=062427299064
AWS_DEFAULT_REGION=ca-central-1
AWS_ECR_REGISTRY=${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}

# Git submodules
src-update-submodules: ## Update submodules to match their version
	git submodule sync --recursive
	git submodule update --init --recursive

# ROS

.PHONY: build
WS_SRC?=.
build: ## Build the project in debug mode
	/bin/bash -c ' \
		. /opt/ros/${ROS_DISTRO}/setup.bash && \
		cd ${WS_SRC} && \
		catkin_make && \
		. devel/setup.bash'	

.PHONY: build-robot-workspace
build-robot-workspace: ## Build the robot workspace in debug mode
	/bin/bash -c ' \
		. /opt/ros/${ROS_DISTRO}/setup.bash && \
		cd robot_ws && \
		catkin_make && \
		. devel/setup.bash'		

.PHONY: build-rs
build-rs: build-robot-workspace build-simulation-workspace

build-release: ## Build the project in release mode
	/bin/bash -c ' \
		. /opt/ros/${ROS_DISTRO}/setup.bash && \
		cd robot_ws && \
		colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release --install-base /opt/${PROJECT}'

build-sim-release: ## Build the project in release mode
	/bin/bash -c ' \
		. /opt/ros/${ROS_DISTRO}/setup.bash && \
		cd simulation_ws && \
		colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release --install-base /opt/${PROJECT}_sim'

.PHONY: clean-robot-ws
clean-robot-ws: ## Clean install, log and build dirs from the workspace
		@echo "##########################"
		@echo "Cleaning up robot_ws build artifacts"
		cd robot_ws && \
		rm -rf build install log
		
.PHONY: clean-simulation-ws
clean-simulation-ws: ## Clean install, log and build dirs from the workspace
		@echo "##########################"
		@echo "Cleaning up simulation_ws build artifacts"
		cd simulation_ws && \
		rm -rf build install log

.PHONY: clean-rs
## Clean install, log and build dirs from the workspace
clean-rs: clean-robot-ws clean-simulation-ws 

.PHONY: clean
clean: ## Clean install, log and build dirs from the workspace
		@echo "##########################"
		@echo "Cleaning up  ${PROJECT_DIR}"
		rm -rf build install log
		@echo "Cleaning up  ${PROJECT_DIR} submodules"
		git submodule foreach rm -rf build install log

# Autoware

.PHONY: build-autoware
build-autoware: ## Build the project in debug mode
	/bin/bash -c ' \
		. /opt/ros/${ROS_DISTRO}/setup.bash && \
		. ${AUTOWARE_DIR}/setup.bash && \
		cd ${WS_SRC} && \
		colcon build --cmake-args -DCMAKE_BUILD_TYPE=Debug && \
		. install/setup.bash'	

.PHONY: build-robot-workspace-autoware
build-robot-workspace-autoware: ## Build the robot workspace in debug mode
	/bin/bash -c ' \
		. /opt/ros/${ROS_DISTRO}/setup.bash && \
		. ${AUTOWARE_DIR}/setup.bash && \
		cd robot_ws && \
		colcon build --cmake-args -DCMAKE_BUILD_TYPE=Debug && \
		. install/setup.bash'	

.PHONY: build-simulation-workspace-autoware
build-simulation-workspace-autoware: ## Build the robot workspace in debug mode
	/bin/bash -c ' \
		. /opt/ros/${ROS_DISTRO}/setup.bash && \
		. ${AUTOWARE_DIR}/setup.bash && \
		cd simulation_ws && \
		colcon build --cmake-args -DCMAKE_BUILD_TYPE=Debug && \
		. install/setup.bash'	

.PHONY: build-rs-autoware
build-rs-autoware: build-robot-workspace-autoware build-simulation-workspace-autoware

build-release-autoware: ## Build the project in release mode
	/bin/bash -c ' \
		. /opt/ros/${ROS_DISTRO}/setup.bash && \
		. ${AUTOWARE_DIR}/setup.bash && \
		cd robot_ws && \
		colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release --install-base /opt/${PROJECT}'

build-sim-release-autoware: ## Build the project in release mode
	/bin/bash -c ' \
		. /opt/ros/${ROS_DISTRO}/setup.bash && \
		. ${AUTOWARE_DIR}/setup.bash && \
		cd simulation_ws && \
		colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release --install-base /opt/${PROJECT}_sim'

# Docker

.upd-params: ## Set docker image names based on environment variables
	$(eval SUBBASE_IMAGE_NAME=${IMAGE_REPO_NAME}:${PROJECT}-subbase-${TAG})
	$(eval SUBBASE_IMAGE_NAME_AWS_ECR = ${AWS_ECR_REGISTRY}:${PROJECT}-subbase-${TAG})
	$(eval SUBBASE_IMAGE_NAME_DOCKERHUB = ${DOCKERHUB_REGISTRY}:${PROJECT}-subbase-${TAG})
	$(eval BASE_IMAGE_NAME=${IMAGE_REPO_NAME}:${PROJECT}-base-${TAG})
	$(eval BASE_IMAGE_NAME_AWS_ECR = ${AWS_ECR_REGISTRY}:${PROJECT}-base-${TAG})
	$(eval BASE_IMAGE_NAME_DOCKERHUB = ${DOCKERHUB_REGISTRY}:${PROJECT}-base-${TAG})
	$(eval DEV_IMAGE_NAME=${IMAGE_REPO_NAME}:${PROJECT}-dev-${TAG})
	$(eval DEV_IMAGE_NAME_AWS_ECR = ${AWS_ECR_REGISTRY}:${PROJECT}-dev-${TAG})
	$(eval DEV_IMAGE_NAME_DOCKERHUB = ${DOCKERHUB_REGISTRY}:${PROJECT}-dev-${TAG})
	$(eval PROD_IMAGE_NAME=${IMAGE_REPO_NAME}:${PROJECT}-prod-${TAG})
	$(eval PROD_IMAGE_NAME_AWS_ECR = ${AWS_ECR_REGISTRY}:${PROJECT}-prod-${TAG})
	$(eval PROD_IMAGE_NAME_DOCKERHUB = ${DOCKERHUB_REGISTRY}:${PROJECT}-prod-${TAG})

docker-login:
	aws ecr get-login-password --region ca-central-1 | docker login --username AWS --password-stdin 062427299064.dkr.ecr.ca-central-1.amazonaws.com

.build-docker-subbase: ## Build the docker subbase image
	docker build -f binder/Dockerfile.subbase \
	-t ${SUBBASE_IMAGE_NAME} \
	-t ${SUBBASE_IMAGE_NAME_AWS_ECR} \
	--build-arg AUTOWARE_VER=${AUTOWARE_VER} \
	--build-arg ROS_DISTRO=${ROS_DISTRO} \
	--build-arg PROJECT=${PROJECT} \
	--build-arg TAG=${TAG} \
	.

.build-docker-base: ## Build the docker base image
	docker build -f binder/Dockerfile.base \
	-t ${BASE_IMAGE_NAME} \
	-t ${BASE_IMAGE_NAME_AWS_ECR} \
	--build-arg AUTOWARE_VER=${AUTOWARE_VER} \
	--build-arg ROS_DISTRO=${ROS_DISTRO} \
	--build-arg PROJECT=${PROJECT} \
	--build-arg TAG=${TAG} \
	--build-arg IMAGE_MODE=${IMAGE_MODE} \
	.

.build-docker-dev: ## Build the docker dev image
	docker build -f binder/Dockerfile.dev \
	-t ${DEV_IMAGE_NAME} \
	-t ${DEV_IMAGE_NAME_AWS_ECR} \
	--build-arg BASE_IMAGE=${BASE_IMAGE_NAME} \
	--build-arg PROJECT=${PROJECT} \
	.

.build-docker-prod: ## Build the docker prod image
	docker build -f binder/Dockerfile.prod \
	-t ${PROD_IMAGE_NAME} \
	-t ${PROD_IMAGE_NAME_AWS_ECR} \
	--build-arg BASE_IMAGE=${BASE_IMAGE_NAME} \
	--build-arg DEV_IMAGE=${DEV_IMAGE_NAME} \
	--build-arg PROJECT=${PROJECT} \
	.

docker-build-subbase: .upd-params .build-docker-subbase	## build subbase image
docker-build-base: .upd-params .build-docker-base	## build base image
docker-build-dev: .upd-params .build-docker-dev  ## build dev image
docker-build-prod: .upd-params .build-docker-prod  ## build prod cpu image

docker-test-prod: .upd-params ## Run simple test on the local image and check for the required ros nodes and topics
	echo "Running tests"
	docker run --rm -v ${PWD}/tests/:/tmp/tests/ --entrypoint bash  ${PROD_IMAGE_NAME} -c 'cd /tmp/tests && bash run_tests.bash'	
	echo "Tests finished successfuly"
	@sleep 4
	docker ps -a

docker-push-subbase: .upd-params 
	docker push ${SUBBASE_IMAGE_NAME_AWS_ECR}
#	docker push ${SUBBASE_IMAGE_NAME_DOCKERHUB}

docker-push-base: .upd-params 
	docker push ${BASE_IMAGE_NAME_AWS_ECR}
#	docker push ${BASE_IMAGE_NAME_DOCKERHUB}

docker-push-dev: .upd-params 
	docker push ${DEV_IMAGE_NAME_AWS_ECR}
#	docker push ${DEV_IMAGE_NAME_DOCKERHUB}

docker-push-prod: .upd-params 
	docker push ${PROD_IMAGE_NAME_AWS_ECR}
#	docker push ${PROD_IMAGE_NAME_DOCKERHUB}

docker-pull-base: .upd-params 
	docker pull ${BASE_IMAGE_NAME_AWS_ECR}
	docker tag ${BASE_IMAGE_NAME_AWS_ECR} ${BASE_IMAGE_NAME}

docker-pull-dev: .upd-params 
	docker pull ${DEV_IMAGE_NAME_AWS_ECR}
	docker tag ${DEV_IMAGE_NAME_AWS_ECR} ${DEV_IMAGE_NAME}

docker-pull-jupyter: .upd-params
	docker pull ${AWS_ECR_REGISTRY}:jupyter-ros-latest
	docker tag ${AWS_ECR_REGISTRY}:jupyter-ros-latest jupyter-ros

docker-prune:
	docker system prune --all --force

docker-remove-no-tags: ## Remove Docker images with no tags
	docker rmi $$(docker images -f dangling=true -q)

docker-remove-containers:
	docker rm $$(docker ps -a -q)

# ROS builds
ros-dep-install: ## Install ros dependencies for the target $ROS_DISTRO
	/bin/bash -c ' \
		. /opt/ros/${ROS_DISTRO}/setup.bash && \
		apt-get update && \
		cd /opt/ros/${ROS_DISTRO}/ && \
		sudo rosdep fix-permissions && \
		rosdep update --rosdistro ${ROS_DISTRO} && \
		rosdep install --default-yes --from-paths . --ignore-src --rosdistro ${ROS_DISTRO} && \
		rm -rf /var/lib/apt/lists/* /tmp/apt-packages'	

rosdep-install-user: ## Install ros dependencies for the target $ROS_DISTRO as $USER
	/bin/bash -c ' \
		. /opt/ros/${ROS_DISTRO}/setup.bash && \
		sudo apt-get update && \
		sudo rosdep fix-permissions && \
		rosdep update --rosdistro ${ROS_DISTRO} && \
		rosdep install --default-yes --from-paths . --ignore-src --rosdistro ${ROS_DISTRO} && \
		sudo rm -rf /var/lib/apt/lists/* /tmp/apt-packages'	

rosnode-kill:
	rosnode kill -a; killall -9 rosmaster; killall -9 roscore

docker-stop-base: .upd-params
	docker stop ${PROJECT}-binder-base || true && docker rm ${PROJECT}-binder-base || true

docker-stop-dev: .upd-params
	docker stop ${PROJECT}-binder-dev|| true && docker rm ${PROJECT}-binder-dev || true

docker-start-base: .upd-params docker-stop-base
	docker run \
	-h ${PROJECT}-binder-base \
	--detach \
	--name ${PROJECT}-binder-base \
	--env COLORFGBG \
	--env EMAIL \
	--env GIT_AUTHOR_EMAIL \
	--env GIT_AUTHOR_NAME \
	--env GIT_COMMITTER_EMAIL \
	--env GIT_COMMITTER_NAME \
	--env SSH_AUTH_SOCK \
	--env TERM \
	--env USER=${USER} \
	--env GROUP=${USER} \
	--env USER_ID=`id -u ${USER}` \
	--env GROUP_ID=`getent group ${USER} | awk -F: '{printf $$3}'` \
	-v /dev:/dev \
	-v /tmp/.X11-unix:/tmp/.X11-unix \
	-v $${PWD}:/home/${USER} \
	--env DISPLAY \
	--env NVIDIA_VISIBLE_DEVICES=all \
	--gpus all \
	--env NVIDIA_DRIVER_CAPABILITIES=compute,utility,graphics,display \
	--env LD_LIBRARY_PATH=/usr/local/nvidia/lib64 \
	--env VIDEO_GROUP_ID=`getent group video | awk -F: '{printf $$3}'` \
	-v /dev/dri:/dev/dri \
	-v /home/${USER}/.ssh:/home/${USER}/.ssh \
	-v /run/user/`id -u ${USER}`/keyring/ssh:/run/user/`id -u ${USER}`/keyring/ssh \
	--cap-add=SYS_PTRACE \
	--net=host \
	--privileged \
	--add-host ${PROJECT}-binder-base:127.0.0.1 \
	${BASE_IMAGE_NAME}
	xhost +local:'${PROJECT}-binder-base'

docker-start-dev: .upd-params docker-stop-dev
	docker run \
	-h ${PROJECT}-binder-dev \
	--detach \
	--name ${PROJECT}-binder-dev \
	--env COLORFGBG \
	--env EMAIL \
	--env GIT_AUTHOR_EMAIL \
	--env GIT_AUTHOR_NAME \
	--env GIT_COMMITTER_EMAIL \
	--env GIT_COMMITTER_NAME \
	--env SSH_AUTH_SOCK \
	--env TERM \
	--env USER=${USER} \
	--env GROUP=${USER} \
	--env USER_ID=`id -u ${USER}` \
	--env GROUP_ID=`getent group ${USER} | awk -F: '{printf $$3}'` \
	--env-file ./binder/phoenix1-fleet-management/${ROBOT_NAME}.env \
	-v /dev:/dev \
	-v /tmp/.X11-unix:/tmp/.X11-unix \
	-v $${PWD}:/home/${USER} \
	--env DISPLAY \
	--env NVIDIA_VISIBLE_DEVICES=all \
	--gpus all \
	--env NVIDIA_DRIVER_CAPABILITIES=compute,utility,graphics,display \
	--env LD_LIBRARY_PATH=/usr/local/nvidia/lib64 \
	--env VIDEO_GROUP_ID=`getent group video | awk -F: '{printf $$3}'` \
	-v /dev/dri:/dev/dri \
	-v /home/${USER}/.ssh:/home/${USER}/.ssh \
	-v /run/user/`id -u ${USER}`/keyring/ssh:/run/user/`id -u ${USER}`/keyring/ssh \
	--cap-add=SYS_PTRACE \
	--net=host \
	--privileged \
	--add-host ${PROJECT}-binder-dev:127.0.0.1 \
	${DEV_IMAGE_NAME}
	xhost +local:'${PROJECT}-binder-dev'

docker-start-root: .upd-params docker-stop-dev 
	docker run \
	-h ${PROJECT}-binder-dev \
	--detach \
	--name ${PROJECT}-binder-dev \
	--env COLORFGBG \
	--env TERM \
	-v /dev/shm:/dev/shm \
	-v /tmp/.X11-unix:/tmp/.X11-unix \
	--env DISPLAY \
	--env VIDEO_GROUP_ID=`getent group video | awk -F: '{printf $$3}'` \
	--volume /etc/timezone:/etc/timezone:ro \
	--volume /etc/localtime:/etc/localtime:ro \
	-v /dev/dri:/dev/dri \
	--cap-add=SYS_PTRACE \
	--net=host \
	--privileged \
	--add-host ${PROJECT}-binder-dev:127.0.0.1 \
	--volume $${PWD}/binder/.ais:/root/.ais \
	${PROD_IMAGE_NAME} 
	xhost +local:'${PROJECT}-binder-dev'

docker-start-jupyter:
	docker run \
	-h jupyter-ros \
	--name jupyter-ros \
	--env COLORFGBG \
	--env EMAIL \
	--env GIT_AUTHOR_EMAIL \
	--env GIT_AUTHOR_NAME \
	--env GIT_COMMITTER_EMAIL \
	--env GIT_COMMITTER_NAME \
	--env SSH_AUTH_SOCK \
	--env TERM \
	--env USER=${USER} \
	--env GROUP=${USER} \
	--env USER_ID=`id -u ${USER}` \
	--env GROUP_ID=`getent group ${USER} | awk -F: '{printf $$3}'` \
	-v /dev:/dev \
	-v /tmp/.X11-unix:/tmp/.X11-unix \
	-v $${PWD}:/home/${USER} \
	--env DISPLAY \
	--env NVIDIA_VISIBLE_DEVICES=all \
	--gpus all \
	--env NVIDIA_DRIVER_CAPABILITIES=compute,utility,graphics,display \
	--env LD_LIBRARY_PATH=/usr/local/nvidia/lib64 \
	--env VIDEO_GROUP_ID=`getent group video | awk -F: '{printf $$3}'` \
	-v /dev/dri:/dev/dri \
	-v /home/${USER}/.ssh:/home/${USER}/.ssh \
	-v /run/user/`id -u ${USER}`/keyring/ssh:/run/user/`id -u ${USER}`/keyring/ssh \
	--cap-add=SYS_PTRACE \
	--net=host \
	--privileged \
	--add-host jupyter-ros:127.0.0.1 \
	${AWS_ECR_REGISTRY}:jupyter-ros-latest

docker-enter-base: .upd-params
	docker exec -it --env COLORFGBG --env TERM -u ${USER} --workdir /home/${USER} ${PROJECT}-binder-base /bin/bash -li

docker-enter-dev: .upd-params
	docker exec -it --env COLORFGBG --env TERM -u ${USER} --workdir /home/${USER} ${PROJECT}-binder-dev /bin/bash -li

docker-enter-base-root: .upd-params
	docker exec -it --env COLORFGBG --env TERM --workdir /root ${PROJECT}-binder-base /bin/bash -li

docker-enter-dev-root: .upd-params
	docker exec -it --env COLORFGBG --env TERM --workdir /root ${PROJECT}-binder-dev /bin/bash -li

docker-start-sim: .upd-params docker-stop-dev
	mkdir -p $${PWD}/.ais/logs && mkdir -p $${PWD}/.ais/maps
	docker run --privileged -h  ${PROJECT}-binder-dev --name ${PROJECT}-binder-dev -d --cap-add=SYS_PTRACE \
	   --net=host \
	   --add-host ${PROJECT}-binder-dev:127.0.0.1 \
	   --env TERM \
	   --env DISPLAY \
	   --env VIDEO_GROUP_ID=`getent group video | awk -F: '{printf $$3}'` \
	   --env-file ./binder/simulation.env \
	   --volume /dev/dri:/dev/dri \
	   --volume /dev/input:/dev/input \
	   --gpus all \
	   --env NVIDIA_VISIBLE_DEVICES=all \
	   --env NVIDIA_DRIVER_CAPABILITIES=compute,utility,graphics,display \
	   --env LD_LIBRARY_PATH=/usr/local/nvidia/lib64 \
	   --volume $${PWD}/.ais:/root/.ais \
	   ${DEV_IMAGE_NAME} bash -c 'source /tmp/run_simulation.sh'
	xhost +local:'${PROJECT}-binder-dev'

bring-up: .upd-params
	mkdir -p $${PWD}/.ais/logs && mkdir -p $${PWD}/.ais/maps
	docker run --privileged -h  ${PROJECT}-binder-prod --name ${PROJECT}-binder-prod -d --cap-add=SYS_PTRACE \
	   --net=host \
	   --add-host ${PROJECT}-binder-prod:127.0.0.1 \
	   --env TERM \
	   --env DISPLAY \
	   --env VIDEO_GROUP_ID=`getent group video | awk -F: '{printf $$3}'` \
	   --env-file ./binder/simulation.env \
	   --volume /dev/dri:/dev/dri \
	   --volume /dev/input:/dev/input \
	   --gpus all \
	   --env NVIDIA_VISIBLE_DEVICES=all \
	   --env NVIDIA_DRIVER_CAPABILITIES=compute,utility,graphics,display \
	   --env LD_LIBRARY_PATH=/usr/local/nvidia/lib64 \
	   --volume $${PWD}/.ais:/root/.ais \
	   ${PROD_IMAGE_NAME}
	xhost +local:'${PROJECT}-binder-prod'

run: ## Run the project inside the ADE
	/bin/bash -c '${PROJECT_DIR}/build-resources/entrypoints/docker-entrypoint.bash'
	/bin/bash -c 'web_script_launch.bash'
	/bin/bash -c 'web_script_pub.bash'

.PHONY: help

help: ## Print help for the make targets
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort |  sed -En "s/^[a-zA-Z_-]+:([a-zA-Z_-]+:.*?## .*)/\1/p" | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-50s\033[0m %s\n\n", $$1, $$2}'

run-mission-cmd:
	/bin/bash -c ' \
		source web_script_launch.sh'

pub-mission-cmd:  
	/bin/bash -c ' \
		source web_script_pub.sh'

rosdep-keys:
	rosdep keys --from-paths . -i  --rosdistro ${ROS_DISTRO}

rosdep-keys-install:
	rosdep keys --from-paths . --ignore-src -n --rosdistro ${ROS_DISTRO} |  xargs echo -n | xargs rosdep install --rosdistro ${ROS_DISTRO} 

rosdep-keys2apt:
	rosdep keys --from-paths . --ignore-src -n --rosdistro ${ROS_DISTRO} | xargs echo -n

GPG: # Update the GPG key of ROS Melodic
	curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -
	sudo apt update
