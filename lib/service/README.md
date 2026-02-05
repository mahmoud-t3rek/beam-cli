# Service (Behaviour Layer)
Bridge layer between the CLI commands and the js-behaviours core execution engine.

## Overview

The Service layer provides a typed, Observable-based API that the CLI uses to discover, run, and track backend behaviours.
It keeps the command handlers thin by centralizing execution, parameter resolution, caching, and error handling inside services.

Architecture
text
   [ CLI  ]
      ↓
[ RequestsService (Registry/Cache) ]  ⇄  [ BehaviourService (Orchestrator) ]
      ↓
[ js-behaviours (Core Execution Engine) ]
      ↓
[ Backend API / WebSocket ]


### Key Concepts

- **js-behaviours**: Core engine that discovers behaviours and returns executable handlers.
- **RequestsService**: CLI state/cache manager; loads the behaviours list, stores parameter drafts, caches responses, and exposes “current run” metadata.
- **BehaviourService**: CLI executor; resolves a behaviour runner by name and executes it with parsed CLI parameters, streaming success/error back to the store.

---

### Initialization (Factory Provider)

Goal: create a singleton Behaviours instance using runtime CLI config (baseURL + prefix).

```ts
export function getBehaviours(http: HttpClient): Behaviours {
  const config = inject(TEST_BEHAVIOURS_UI_CONFIG);

  const fullURL = config.baseURL
    ? new URL(config.prefix, config.baseURL).href
    : config.prefix;

  return new Behaviours(http, fullURL);
}


RequestsService (Discovery + State)
Responsibilities

Wait for engine readiness before running any CLI command.

Fetch behaviour registry from /behaviours (powers beam list and validation for beam run <name>).

Map the backend dictionary into a CLI-friendly array (or Map) for fast lookup.

Maintain state for CLI output and tooling: loading, requests list, active request, last response, last error, execution timing.

Persist drafts and cached results using a CLI-appropriate store (filesystem cache recommended for Node CLIs; localStorage only applies to browser contexts).

Example
ts

this.behaviours.ready(() => {
  this.behaviours.behaviours({}).subscribe({
    next: (res) => {
      this.requests.set(Object.keys(res).map(name => ({ name, ...res[name] })));
    },
    error: () => this.requests.set([])
  });
});


BehaviourService (Execution)
Responsibilities
Resolve behaviour runner: behaviours.getBehaviour(name).

Inject parameters parsed from CLI flags/stdin/config file.

Execute the behaviour (Observable) and handle subscription lifecycle.

Push success/error + execution timing back into RequestsService.

Return/print formatted output suitable for terminals (pretty JSON, raw output, or quiet mode), and set exit codes appropriately.

Example



send(requestName: string): void {
  const fn = this.behaviours.getBehaviour(requestName);

  fn(this.parametersSignal()).subscribe({
    next: (response) => this.updateResponse(response),
    error: (error) => this.errorSignal.set(error)
  });
}

CLI Capabilities
List behaviours: Auto-scan backend (/behaviours) and display available commands (e.g., beam list).

Run behaviour: Execute a behaviour by name (e.g., beam run <name>) with dynamic parameter resolution (path/query/header/body).

Caching: Store responses and parameter drafts to speed up repeated calls and enable offline inspection.

Exports (optional): Support file output (e.g., --out result.json) for scripting and pipelines.

WebSocket (optional): Support event-based behaviours and stream events to stdout.

Configuration
ts

export interface BehavioursConfig {
  baseURL: string; // e.g. http://localhost:3000
  prefix: string;  // e.g. /api
}
