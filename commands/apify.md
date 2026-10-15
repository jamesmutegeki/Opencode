---
description: Route an Apify task (scraping, Actors, SDK integration) through the apify subagent
agent: apify
subtask: true
---

Use the Apify subagent to handle this request. It owns the Apify routing matrix and will dispatch to the right skill, MCP tool, CLI workflow, or SDK integration for:
- using existing Actors (scraping, browser automation, data extraction),
- building, testing, or deploying custom Actors,
- adding Apify to an existing application via apify-client / the Apify SDK.
