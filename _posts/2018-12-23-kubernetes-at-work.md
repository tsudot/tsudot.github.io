---
layout: post
title: Kubernetes at Work
location: Bangalore
---

Original Post: https://blog.karix.io/how-we-build-and-run-karix.io

This post describes how we run and manage karix.io on kubernetes, using helm for packaging and jenkins for CI/CD 
Summary
 
This post is part one of a series of posts that describe how we built and now run karix.io. It covers the entire engineering effort undertaken and how and why we got here based on past learnings and experiences. It is my journey from a docker-compose PoC to kubernetes in production with the primary goal of improving developer/ops productivity and as a result ensuring a high uptime and quality of service for our customers.
 
## Background
 
Two years ago, the team I was on at a previous organization, was tasked with building the tech platform for an entirely new business unit of the company. It had to integrate with all existing services and have a presence on both web and mobile.
 
I took on the responsibility of enabling this effort by providing developer tooling for the team. We were building a new product, the goal was to setup a testing environment for each component of the stack and also the entire service as a whole. Back then, we were running all of our code on chunky VMs. The process to do integration testing was to deploy code to a staging environment which was setup using ansible.
 
We had only one environment to do QA and acceptance testing for all teams, working across locations! This was a big hurdle and a major bottleneck in terms of developer productivity. We soon realised that we could either go on with a single QA/staging environment or meet deadlines, not both. The goal was clear, we couldn’t timeshare the test environment. We needed to have one when needed. Bonus if we could make it available on-demand for as many developers in parallel.
 
## Hello Containers
 
I started using docker swarm to run unit tests locally (per component). There were many components for which a Dockerfile wasn’t present. This was not a major problem since most applications were written in Python or Golang and it was fairly easy to write Dockerfiles for each of these components. This effort started with infiltrating every repo with a “containerize” branch. Config was hardcoded in each repository under different folders for specific environments (staging/production). This was a conscious decision taken to get things up and running fast. This was known technical debt we were picking up and a proper config and secret management strategy would need to be planned later.
 
Eventually we had a system running on docker-compose (for developers to work and test on their local machines) and kubernetes in staging for QA/acceptance testing.  While we never got to containerising the production environment, we were able to ensure we went past the hurdle of a shared staging environment and as a result were able to meet business deadlines for the product release.
 
## Kubernetes and karix.io
 
When I started working on karix.io last year, we were to build a global self-service API platform for two way messaging. We first worked on the architecture, deciding on what microservices to build to ensure we had isolated and independently scalable components for separate functions like authentication, billing, routing etc. Once that was done, based on my previous learnings I put forward the idea of using kubernetes to orchestrate our services and resources.
 
At the time, we were only two engineers working on this. The thought process was to make sure that most developer time is spent on building the service and not managing infrastructure. Also to build a system which would ensure super simple onboarding of new developers as the team grew. Based on this, we decided to use managed services for critical components which could be single point of failures. We went with Google Cloud as we could use GKE (Google Kubernetes Engine) to manage our kubernetes master along with other google managed services like CloudSQL for our data stores.
 
We decided to create 3 kubernetes clusters to start with, first one was the CI cluster; this cluster is used to host jenkins and other build tools. The other 2 clusters were staging and production. We started out by making guidelines to follow when writing any new component on the platform. The most important one being that every repository had to have Jenkinsfile and a Dockerfile. Jenkinsfile is a text file that contains the definition of a Jenkins Pipeline and is checked into source control. Check out our current guidelines here
 
## Building the pipeline
 
![build pipeline](https://blog.karix.io/hs-fs/hubfs/Screen%20Shot%202018-08-17%20at%201.32.33%20PM.png?width=507&name=Screen%20Shot%202018-08-17%20at%201.32.33%20PM.png)

(image courtesy: docker.com) 
 
On a very abstract level, kubernetes is basically cluster orchestration using containers as highly available deployable units along with (mostly) out of the box service discovery.  
 
So the first step was to have a build pipeline of version controlled, deployable containers along with their associated config for every service/component of the platform and have unit tests as part of safeguards along the build process. The pipeline had to be designed to begin from a developer commit upstream and end at either staging, production or even a random pop-up cluster. It also had to have a super easy way to monitor the steps of the pipeline, slack hooks worked like a charm.
 
We chose jenkins as we didn’t want to experiment with other build systems since I had experience using jenkins before. The only difference this time was that we were running jenkins with the kubernetes plugin. On every new build, jenkins would spawn a new kubernetes pod to run the tests in an isolated contained environment.
 
As stated in the guidelines above a Jenkinsfile was a must for each component, this  would build containers and run tests under a docker network. Data stores’ containers like redis and postgres were attached to the docker network to run unit tests in an isolated network. Build and test scripts were put in a Makefile, which made it pretty straightforward to call them generically from groovy (Jenkinsfile). The build script would then create containers with the release tags and push it to GCR to store our versioned containers for each repository. To summarise, the Jenkinsfile would do the following (in order) and send relevant notifications via slack hooks:
 

- Spawn data store containers under a docker network

- Build the container for that particular repository

- Run unit tests inside the container

- On success, push the container to our internal private registry

- Package the kubernetes config for the repository and push it to our package repository (more on this later)

You can see a trimmed down version of a Jenkinsfile here. This is from a django application that talks to redis and postgres.
Stay Tuned!


Follow up posts in this series will describe how we use helm for packaging kubernetes components, our emphasis on the importance of testing, our tooling for releases/deployments and our take on best practices like monitoring, alerting etc along with future goals. Thanks for reading and stay tuned!
