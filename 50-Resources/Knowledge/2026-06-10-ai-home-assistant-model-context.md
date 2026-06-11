---
type: reference
status: active
created: 2026-06-10
updated: 2026-06-10
source_url: https://www.home-assistant.io/integrations/mcp_server/
source_type: article
source_author: ""
source_title: "Home Assistant Model Context Protocol Server Integration"
source_published: ""
source_duration: "5 min read"
topics: [ai, tech, automation]
summary: "The Home Assistant Model Context Protocol Server (MCP) integration allows Large Language Model (LLM) client applications to receive context from and control Home Assistant devices. This enables LLMs like Claude Desktop or ChatGPT to interact with your smart home setup, providing a rich, real-time understanding of your home's state."
key_points:
  - "The Model Context Protocol (MCP) standardizes how applications provide context to LLMs."
  - "Home Assistant's MCP Server integration allows LLM clients to access Home Assistant's Assist API, enabling control of devices and access to entity states."
  - "Configuration involves adding the MCP Server integration to Home Assistant and then setting up the LLM client (e.g., Claude for Desktop, ChatGPT) with the appropriate server URL and authentication."
  - "Supports OAuth for authorization, utilizing Home Assistant's Authentication API, and also allows for Long-Lived Access Tokens for clients without OAuth support."
  - "Provides options for connecting to Home Assistant instances that are publicly accessible (via remote connector) or only locally accessible (via a local MCP proxy server like `mcp-proxy`)."
relevance: 3
access_count: 0
last_accessed: ""
ingested_by: kb-ingest.py
kb_id: kb-20260610-a78f59
---

# Home Assistant Model Context Protocol Server Integration

## Summary
The Home Assistant Model Context Protocol Server (MCP) integration allows Large Language Model (LLM) client applications to receive context from and control Home Assistant devices. This enables LLMs like Claude Desktop or ChatGPT to interact with your smart home setup, providing a rich, real-time understanding of your home's state.

## Key Points
- The Model Context Protocol (MCP) standardizes how applications provide context to LLMs.
- Home Assistant's MCP Server integration allows LLM clients to access Home Assistant's Assist API, enabling control of devices and access to entity states.
- Configuration involves adding the MCP Server integration to Home Assistant and then setting up the LLM client (e.g., Claude for Desktop, ChatGPT) with the appropriate server URL and authentication.
- Supports OAuth for authorization, utilizing Home Assistant's Authentication API, and also allows for Long-Lived Access Tokens for clients without OAuth support.
- Provides options for connecting to Home Assistant instances that are publicly accessible (via remote connector) or only locally accessible (via a local MCP proxy server like `mcp-proxy`).

## Notable
> This gives your AI assistant a clear picture of your home’s current state.

## My Notes
<!-- Add personal notes here -->

## Source
[Home Assistant Model Context Protocol Server Integration](https://www.home-assistant.io/integrations/mcp_server/) |  | 
