# main Controller

## Prompt 
This library encapsulates the logic for managing the backend processes.

### Core Responsibilities:
1.  **Clustering**: Implement clustering logic to spawn backend workers.
2.  **Health Check**: Implement a "Wait" mechanism to ensure the backend process is live before sending requests.
3.  **Path Management**: It must accept the path to the backend `server.js` and the port from environment variables.
