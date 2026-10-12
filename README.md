# @tannerbroberts/about-time-mcp

An MCP server that lets an AI agent turn what a person wants to do into a **plan they can actually follow**. A plan here is:

- nested into steps
- ordered and timed
- scaled to the number of people
- scheduled around what one person can do at once
- checked against their allergies, budget, nutrition targets and equipment
- able to produce a shopping list

```json
{ "mcpServers": { "about-time": { "command": "npx", "args": ["-y", "github:tannerbroberts/about-time-mcp"] } } }
```

State is stored in `$ABOUT_TIME_HOME`, or `./.about-time` if that is not set.

The agent-facing guide ships with the server and is available through the `guide` tool. Agents should start with `guide("overview")`.

Pin a version with `github:tannerbroberts/about-time-mcp#v0.1.0-alpha.3`. This repository holds built releases only.
