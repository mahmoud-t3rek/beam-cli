# main Controller
Create main contoller ts/js

## Prompt 
This controller encapsulates the logic for managing the backend processes and command strategy.

### Core Responsibilities:
1.  **Clustering**: Implement clustering logic to spawn backend child processes if in main process is the current process then it should go to step 2 but if the current process is a child process then it should go to step 2
2.  **Health Check**: if the requested command is local command then to delegate it to resposible controller/s and service/s or if the requested command not local command then It should read env var BASE_URL and connect to it if valid through backend service then in callback of after await it should call the requested command through backend service and wait for the response to be shown fomatted to the user in the console. if BASE_URL not valid then start the backend process from local backend folder and implement a "Wait" mechanism to ensure the backend process is live by calling the health check endpoint through backend service passing static base url as localhost:[PORT or default 3000] and prefix as defined in local backend then when it is live call the requested command through backend service and wait for the response to be shown fomatted to the user in the console.

## Basic Cluster Setup

const cluster = require('cluster');

if (cluster.isPrimary) {
  console.log(`Master ${process.pid} is running`);

  cluster.fork();

  cluster.on('exit', (worker) => {
    console.log(`Worker ${worker.process.pid} died. Restarting...`);
    cluster.fork();
  });

  .......

} else {
 .......
}
