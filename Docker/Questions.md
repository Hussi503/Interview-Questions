
## 🔴 1. Docker images vs Docker containers - what's the difference?
A **Docker image** is a read-only blueprint or template that contains everything required to run an application, such as application code, runtime, libraries, dependencies, and configurations.
A **Docker container** is the actual running instance of that image.

| Docker Image | Docker Container |
|-------------|------------------|
| Blueprint of the application | Running instance of the application |
| Immutable | Ephemeral and runtime-based |
| Stored in Registry | Runs on Docker/Kubernetes host |
| Built using Dockerfile | Created from an image |
| Versioned artifact | Consumes CPU and Memory |

## 🔴2. What are the advantages of containerizing applications?

The biggest advantage of containerizing applications is consistency.

A container packages the application code, runtime, libraries, and dependencies together, so the application behaves the same across a developer's machine, testing environments, and production.

This eliminates the classic 'works on my machine' problem.

Containers are also lightweight compared to virtual machines because they share the host operating system kernel. This results in faster startup times, better resource utilization, and higher application density on servers.

Another major advantage is portability. The same container image can run on any environment that supports Docker or Kubernetes, whether it's on-premises, AWS, Azure, or GCP. This makes migrations and multi-cloud deployments much easier.

## 🔴 3. What is a Dockerfile and what instructions do you use in it?

A Dockerfile is a text file that contains a set of instructions used to automatically build a Docker image.

Instead of manually installing dependencies and configuring environments, we define everything as code in a Dockerfile, which ensures consistency and repeatability across all environments.

  **FROM** : Defines the base image.
  
  **WORKDIR**: Sets the working directory inside the container.
  
  **COPY** : Copies files from host to container.
  
  **ADD** : Copies files and supports URL download and archive extraction.
  
  **RUN**:  Executes commands during image build.
  
  **ENV** :Sets environment variables.
  
  **ARG** : Defines build-time variables.
  
  **EXPOSE** :Documents the port used by the application.
  
  **USER** :Runs the container as a non-root user.
  
  **CMD** : Provides the default command executed when the container starts.
  
  **ENTRYPOINT** :Defines the main executable of the container.

            ```dockerfile
                FROM python:3.11-slim

                WORKDIR /app

                COPY requirements.txt .

                RUN pip install --no-cache-dir -r requirements.txt

                COPY . .

                EXPOSE 5000

                USER 1001

                CMD ["python","app.py"]
              ```
  

## 🔴 4. How do you reduce Docker image size using multi-stage builds?

## 🔴 5. Why should containers run as non-root?

## 🔴 6. What is image immutability?

## 🔴 7. Difference between ADD and COPY in Dockerfile

## 🔴 8. Difference between CMD and ENTRYPOINT - can we use both?

## 🔴 9. What happens if both CMD and ENTRYPOINT are defined?

## 🔴 10. What challenges do you face when containerizing legacy monolithic applications?

## 🔴 11. How do you make a legacy application stateless for containers?

## 🔴 12. How do you handle filesystem dependencies and background jobs in containers?

## 🔴 13. What is the "one process per container" principle?

## 🔴 14. How do sidecars help in containerizing legacy applications?

## 🔴 15. What security considerations do you take while containerizing old applications?

## 🔴 16. How is the Docker client different from the Docker daemon?

## 🔴 17. What is Docker networking and which commands create bridge/overlay networks?

## 🔴 18. What is the difference between Docker, Dockerfile, and Docker Compose?

## 🔴 19. What are three best practices to secure a Docker container?

## 🔴 20. What is the difference between a Dockerfile and a Docker registry?

## 🔴 21. Have you worked with Docker volumes? What happens to a volume if a container is removed?

## 🔴 22. Can you create a Dockerfile for a Python application?

## 🔴 23. If requirements.txt is at a remote location, how will you copy it in Dockerfile?

## 🔴 24. What are the kinds of Docker volumes that are available?

## 🔴 25. Can you tell me, with Docker volume on the host `/app/db` and inside the container it should be mounted to `/opt/data`?
### 🔴 The container name should be `test`. Can you give that command?

## 🔴 26. In Docker, how do you inspect a container?

Narayana
------------------------------
1. What is the difference between a Docker image and a Docker container?
2. What are the advantages of containerizing applications?
3. What is a Dockerfile, and what instructions do you use in it?
4. How do you reduce Docker image size using multi-stage builds?
5. Why should containers run as non-root?
6. What is image immutability?
7. What is the difference between ADD and COPY in a Dockerfile?
8. What is the difference between CMD and ENTRYPOINT? Can you use both?
9. What happens if both CMD and ENTRYPOINT are defined?
10. What challenges do you face when containerizing legacy monolithic applications?
11. How do you make a legacy application stateless for containers?
12. How do you handle filesystem dependencies and background jobs in containers?
13. What is the "one process per container" principle?
14. How do sidecars help in containerizing legacy applications?
15. What security considerations do you take while containerizing old applications?
16. How is the Docker client different from the Docker daemon?
17. What is Docker networking, and which commands create bridge/overlay networks?
18. What is the difference between Docker, Dockerfile, and Docker Compose?
19. What are three best practices to secure a Docker container?
20. What is the difference between a Dockerfile and a Docker registry?
21. Have you worked with Docker volumes? What happens to a volume if the container is
removed?
22. Can you create a Dockerfile for a Python application?
23. If requirements.txt is at a remote location, how will you copy it in a Dockerfile?
24. What is the difference between volumes and bind mounts? When would you use each?
25. What are cgroups and namespaces?
26. What are dangling images and how do you clean them?
27. How do you manage container logs to prevent disk space issues?


DOCKER - The "Container Won't Start" Nightmares
1. Container exits immediately. docker logs shows nothing. How do you debug?
    * Hint: docker inspect, check exit code, override entrypoint to sleep, check permissions
2. Your Docker image is 2GB. How do you reduce to 200MB without breaking the app?
    * Hint: Multi-stage builds, alpine base, clean apt cache, combine RUN layers, .dockerignore
3. Container runs as root. Security team is furious. How do you fix this without breaking the app?
    * Hint: USER directive, --user flag, rootless docker, capability dropping
4. Your application needs to write files to host. Container can't write. Permissions are 777. Still can't write. Why?
    * Hint: SELinux, mount propagation, user namespace remapping
5. docker build works on your laptop but fails on Jenkins agent with "no such file". Why?
    * Hint: Context path differences, files not copied, .dockerignore, symlinks
6. How do you debug why container is using 100% CPU without exec'ing into it?
    * Hint: docker stats, docker top, /sys/fs/cgroup, inspect from host
7. Your container needs to access database on host. How do you connect?
    * Hint: host.docker.internal (Mac/Windows), --network=host, use host IP
8. Docker containers are losing data on restart. How do you persist data properly?
    * Hint: Volumes, bind mounts, named volumes, volume drivers
9. How do you scan Docker images for vulnerabilities in CI/CD pipeline?
    * Hint: Trivy, Clair, Snyk, Docker Scout, fail build on critical
10. Container exits with code 137. What does this mean? How do you fix?
    * Hint: OOMKilled, increase memory limit, optimize memory usage
11. You have 10 microservices that need to talk to each other. How do you orchestrate locally?
    * Hint: Docker Compose, networks, service discovery, depends_on
12. Docker build is slow. Every RUN command invalidates cache. How do you optimize?
    * Hint: Order layers by frequency of change, combine RUN, use cache mounts
13. Your container can't resolve DNS. Host can. What's wrong?
    * Hint: /etc/resolv.conf in container, docker daemon DNS settings, network mode
14. How do you limit container resources (CPU, memory, disk) and why?
    * Hint: --memory, --cpus, --storage-opt, prevent noisy neighbor
15. Container logs are too verbose and filling disk. How do you manage log rotation?
    * Hint: Docker logging drivers, max-size, max-file, json-file options
