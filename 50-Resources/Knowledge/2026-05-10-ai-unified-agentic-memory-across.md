---
type: reference
status: active
created: 2026-05-10
updated: 2026-05-10
source_url: https://towardsdatascience.com/unified-agentic-memory-across-harnesses-using-hooks/
source_type: article
source_author: ''
source_title: Unified Agentic Memory Across Harnesses Using Hooks
source_published: ''
source_duration: ''
topics:
- ai
- tech
- automation
summary: This article proposes a method to create a shared, persistent memory layer
  for AI coding agents across different harnesses (like Claude Code, Codex, and Cursor)
  by leveraging standardized 'hooks'. This approach aims to prevent vendor lock-in
  and ensure deterministic logging of agent session events, independent of the agent's
  decision-making.
key_points:
- Vendor lock-in in AI coding tools is a concern, especially concerning agent memory
  locked within proprietary systems.
- The proposed solution involves keeping the memory layer external to the harness
  and allowing different harnesses to plug into it using hooks.
- Hooks are standardized shell commands that automatically trigger on lifecycle events
  (e.g., session start, prompt submission, tool use) across various AI coding agents.
- Neo4j is used as the persistent store for the shared memory, creating an ordered
  timeline of all session events.
- Hooks enable passive, deterministic logging and the injection of pre-computed memories,
  while Model Context Protocol (MCP) tools provide on-demand agent access to the memory
  layer for searching and updating.
relevance: 3
access_count: 1
last_accessed: '2026-05-10'
ingested_by: kb-ingest.py
kb_id: kb-20260510-05a1ca
---
# Unified Agentic Memory Across Harnesses Using Hooks

## Summary
This article proposes a method to create a shared, persistent memory layer for AI coding agents across different harnesses (like Claude Code, Codex, and Cursor) by leveraging standardized 'hooks'. This approach aims to prevent vendor lock-in and ensure deterministic logging of agent session events, independent of the agent's decision-making.

## Key Points
- Vendor lock-in in AI coding tools is a concern, especially concerning agent memory locked within proprietary systems.
- The proposed solution involves keeping the memory layer external to the harness and allowing different harnesses to plug into it using hooks.
- Hooks are standardized shell commands that automatically trigger on lifecycle events (e.g., session start, prompt submission, tool use) across various AI coding agents.
- Neo4j is used as the persistent store for the shared memory, creating an ordered timeline of all session events.
- Hooks enable passive, deterministic logging and the injection of pre-computed memories, while Model Context Protocol (MCP) tools provide on-demand agent access to the memory layer for searching and updating.

## Notable
> If you don’t own your memory, you don’t own your agent. Every harness today builds its own walled garden of context, preferences, and session history. Switch them and you start from zero. That doesn’t have to be the case.

## My Notes
<!-- Add personal notes here -->

## Source
[Unified Agentic Memory Across Harnesses Using Hooks](https://towardsdatascience.com/unified-agentic-memory-across-harnesses-using-hooks/) |  | 
