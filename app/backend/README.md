# Backend 
create simple beamjs application in js

## Prompt 
This application hosts the core logic and integrates with the BeamJS framework.

### Core Responsibilities:
1.  **Server Entry**: The entry point must be `server.js` (to be created/linked).
2.  **BeamJS Integration**: It will wrap BeamJS logic.
3.  **Process Management**: It will be managed/clustered by the Main Controller.

### Technical Stack:
- Runtime: Node.js
- Framework: BeamJS 

## Entry File

The main entry point of the application is:

./server.js 

This file is responsible for initializing the server and running the application in either single or clustered mode

it reads env var PORT and starts the server on that port or default port 3000
