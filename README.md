# DockerComposeMSADemo
## A Microservice Application deployed to a Docker Compose group on an Azure VM.

## UIDemo
An Angular single page application, designed to send HTTP GET requests to the  jsonplaceholder.typicode.com to-do API, and the HelloApp API. Returned information is displayed for the user.

## HelloApp
A .NET minimal API in C#, designed to return on several endpoints:
- /weatherforecast - returns a set of 5 random weather forecasts
- /count - returns a running request count
- /time - returns the current system time of the API service
- / - returns the string "Hello there! I'm alive!"

## Docker Containers
Both services include a Dockerfile for containerization, and a docker-compose.yml is stored in the root directory of the repository.

## Manual Local Running
Begin by verifying that the services will run independantly in a local environment.
- use `dotnet run` to run the HelloApp api.
- once started, use a browser or HTTP messaging tool (like Postman or ThunderClient) to send GET requests to the HelloApp on `https://localhost:7069` to verify its functionality.

- use `npm run start` to run the UIDemo Angular SPA.
- use a browser to navigate to the running application (port 4200 by default), and use the "Display Posts" button to send a request to the jsonplaceholder.typicode.com API. You should see a list of bullet points displayed as a result of the request and response.

Once independant functionality is verified interaction can be verified. Becuase the .NET SDK and runtime will assign ports to an application at the time of project creation, ports can be modified and edited in launchSettings.json. For this demo the ports are fixed and should be consistent throughout the examples provided.

- use a browser to navigate to the running application (port 4200 by default), and use the "Display Time" button to send a request to the HelloApp service. You should see the current system time of the API displayed as a result of the request and response. You should also see the "Current count is:" message now also displays a value.
- use a browser or HTTP messaging tool (like Postman or ThunderClient) to send GET requests to the HelloApp on `https://localhost:7069/count`. The response from the HellApp should be a value greater than the result last displayed on the DemoApp page.

## Containerized Local Running
Once functionality is confirmed in your local environment, you can begin to containerize the application and services. Each service (the frontend Angular SPA and the backend ASP.NET API) has a Dockerfile included. Creating an image from the Dockerfile is completed using the `docker build` command. If you would like to name the image and give it a tag, you can do so with the `-t` flag. An example of a complete build command is:
`docker build . -t demoimage:latest`

You will need to build images of both services. Remember to name the images differently, as using the same name will overwrite the image you created first. Once both images are built you can use the images to create containers. Use the   `docker run` command to start a container based on each image. Remember to expose the ports that each service will need to communicate outside of its own container. An example of a complete run command is:
`docker run -d -p "80:9999" demoimage:latest`

With both applications containerized and running, perform the same connectivity testing that you did in the previous section, first confirming that the SPA can connect to external resources, confirming that the API can recieve and respond to an HTTP request, and finally that the SPA can make a request to the API.

Use `docker stop {container name/container ID}` commands to stop the running containers. It may be necessary to look up the container names using the `docker ps` or `docker container ls` commands.

## Orchestrated Local Containers
Container orchestration, or controling and managing multiple containers together can be accomplished with Docker Compose. Docker Compose excells at operating in a local environment, and is a common tool for testing an applications services locally before deploying to a larger production environment. Docker Compose uses a declarative YAML format to detail the images, networking, allocations, and other management of the containers to be created. Examine the included `docker-compose.yml` file noting the images, resource requirements, and ports of the different containers. Use the command `docker compose up` to spin up all of the containers listed together. Remember to use the `-d` flag to complete the required action without occupying the terminal. An example of a complete startup command is:
`docker compose up -d`

With the containers running, perform the same connectivity testing as in previous sections to verify full functionality.

Use `docker compose down` to stop the running containers.

## Local Kubernetes Cluster With Minikube
Minikube is a tool that creates a local instance of a Kubernetes (K8s) cluster. The cluster it produces and manages is a small one, limited to the computer that it Minikube is running in, but it will provide us with a local environment for testing before moving our application to a cloud environment.

Visit https://minikube.sigs.k8s.io/docs/start/ for installation instructions. It is recomended that you follow the "Get Started" and "Hello Minikube" tutorials to  become familiar with the minkiube tool, `kubectl` commands, and K8s environment.

Using the images that were built and pushed in previous steps, create a Deployment in a Kubernetes cluster using the `kubectl create` command set. Some specifics can be detailed by the deployment, but most of the things we care about right now will be handled in the next step. An example of a deplolyment command is:
`kubectl create deployment {deployment name} --image={image}`

Verify that the deployment was successful, and that the required pod is created by using `kubectl get` commands. `get pods` and `get deployments` will list all active pods and deployments, but you can specify what deployment or pod you want to list by using `kubectl get pod {pod-name}` or `kubectl get deployment {deployment-name}`

Once you can see the deployment and resulting pod(s), use `kubectl describe pod` or `kubectl describe deployment` with the targets name to display more details on that particular object.

Finally, we can start to connect the running pods to the cluster intranet and external network by exposing the deployments with `kubectl expose` commands. Additional option flags should be used to detail port mapping, type, and service name. For example the command:
`kubectl expose deployment ui --type=NodePort --port=9999 --target-port=80 --name=ui-service`
will create a `NodePort` named `ui-service`. The service will connect to port `80` of each pod, and allow other members of the cluster to contact this service on port `9999`.

Once both ui and api deployments and pods are running, and both are connected to services, we can test each deployment by using a command built into the Minikube environment. Use `minikube service {service-name}` to connect your browser to the services. Verify that both services accept and respond to HTTP requests as expected.

