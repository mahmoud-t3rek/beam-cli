# CLI (NestJS Commander)
create cli ts/js

## Prompt 
This application serves as the primary entry point for user interaction via the terminal.
It must be implemented using **NestJS Commander**.

### Core Responsibilities:
1.  **Command Parsing**: Use `nest-commander` to define commands and sub-commands. there are 3 main commands: 
    - `beam list`: List all available behaviours (remote not local).
    - `beam run <name>`: Run a behaviour by name (remote not local).
    - `beam exec <name> [args...]`: Run a nested command (local command).
    - `beam help`: Display help information (local command).
    - `beam version`: Display version information (local command).
    - .... there scould be other root commands that can have nested commands (local command).
2.  **User Input**: Integrate `inquirer` or `yargs` to collect user flags and interactive inputs.
3.  **Backend Communication**: It should act as a client that triggers processes in the backend or just run the command locally if it is a local command through controllers.

### Technical Stack:
- Framework: NestJS
- Lib: nest-commander
