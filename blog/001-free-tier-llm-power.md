---
title: "What Can You Do With a Free-Tier LLM? Quite a Lot, It Turns Out."
author: "DJ Steve Project"
date: "2026-05-27"
tags: ["llm", "architecture", "efficiency", "free-tier", "gemini", "groq"]
---

# What Can You Do With a Free-Tier LLM? Quite a Lot, It Turns Out.

There's a common assumption in the world of AI today: to build something truly powerful and intelligent, you need access to the biggest, most expensive, state-of-the-art Large Language Models. While these frontier models are incredible, they're not always necessary. With the right architecture, even a free-tier LLM can become the brain of a sophisticated, highly personalized application.

Enter "DJ Steve," a project born from a simple desire: to create Spotify playlists based on my *actual* listening history, not just the whims of a generic algorithm. It's a voice-controlled music assistant that understands requests like "play some punk I haven't heard in a while" or "create a playlist of artists similar to NOFX or Screeching Weasel, but only include tracks I haven't listened to in the last year."

<img src="/about-me/blog/images/01-chat-interface.png" width="600" alt="DJ Steve Chat Interface">

This might sound like it requires a massive model that knows my entire music taste. But it doesn't. The secret lies in a simple but powerful architectural pattern: **the LLM is treated as an orchestrator, not a database.**

## The LLM as a Reasoning Engine

In the DJ Steve architecture, the LLM's only job is to do what it does best: understand natural language and reason about the steps needed to fulfill a request. It doesn't store my listening history, it doesn't know what a "dreamy" song sounds like, and it doesn't directly connect to the Spotify API.

Instead, it has access to a set of over 60 well-defined "tools." These tools are functions exposed by a custom API server (an MCP or "Model Context Protocol" server) that encapsulates all the project's business logic. The heavy lifting—querying a local SQLite database of my listening history, performing fuzzy track deduplication, fetching data from external APIs like Last.fm, and controlling Spotify playback—is all handled by this server.

The LLM's role is simply to be the intelligent switchboard operator, translating a fuzzy human request into a precise sequence of tool calls.

## A Visual Walkthrough: Features of an Orchestrated System

The power of this architecture isn't just in the LLM's reasoning, but in the deterministic features and data transparency that the tool-based system enables. Here’s a look at some of the key features.

### 1. Deterministic Control: Adjusting Playlist Weights

Beyond just a natural language prompt, the UI allows for direct, deterministic control over the playlist creation logic. The user can adjust specific weights—like "Energy" or "Danceability"—that are passed directly to the backend tools. This isn't a fuzzy request; it's a set of precise parameters that the code uses to filter and select tracks. This ensures that the user has fine-grained, reliable control over the output, with the LLM acting as the orchestrator of these well-defined inputs.

<img src="/about-me/blog/images/02-playlist-result.png" width="600" alt="UI for adjusting deterministic weights for playlist creation.">
*The UI allows users to set explicit, deterministic weights that directly influence the backend logic for playlist generation.*

### 2. Local Knowledge: Surfacing Unique Stats

Because the system maintains its own local database of listening history, it can generate and display rich statistics that are impossible to get from the Spotify API alone. This "local knowledge" allows the user to see insights about their own listening patterns, track audio features, and understand the data that the recommendation tools are working with.

<img src="/about-me/blog/images/03-tool-call-example.png" width="600" alt="Display of statistics derived from the local database.">
*The system can surface unique statistics and insights derived from its local knowledge base of the user's listening history.*

### 3. Execution Transparency: The Partial Execution Flow

To build trust and provide visibility, the system doesn't hide its work. For complex queries, it displays a partial execution flow, showing the user the sequence of tools being called to fulfill the request. This transparency reinforces the "LLM as orchestrator" pattern, making it clear that the system is executing a series of discrete, understandable steps.

<img src="/about-me/blog/images/04-spotify-playlist.png" width="600" alt="A partial execution flow showing the sequence of tool calls.">
*The system provides transparency by showing the user the partial execution flow of the tools being run to satisfy their request.*

### 4. The Final Output: Natural Language Response

After all the deterministic logic, tool execution, and data processing is complete, the final step is handed back to the LLM. Its job is to synthesize the results into a concise, easy-to-understand natural language response, confirming that the requested actions have been completed.

<img src="/about-me/blog/images/01-chat-interface.png" width="600" alt="The final natural language response from the LLM.">
*The final step: the LLM provides a clean, natural language summary of the results of the tool execution.*

This entire flow highlights a system that uses an LLM for its strengths (language understanding) while relying on robust, deterministic, and transparent tools for its core logic.

## Why This Architecture is So Efficient

This "LLM as orchestrator" pattern is perfectly suited for the constraints of free-tier models.

*   **Aggressive Cost and Latency Optimization:** The system uses a multi-LLM cascade (Groq → Gemini → Ollama) to stay fast and free. Most requests are handled by Groq's incredibly fast, free-tier Llama 3. If that fails or the request is more complex, it falls back to Gemini Flash, and finally to a locally-hosted Ollama model as a last resort. The result? A power user's daily activity consumes **less than 4% of Groq's free daily quota.**
*   **Small Context Windows:** We never need to stuff the LLM's context with our entire listening history or massive API schemas. The prompt contains only the user's immediate request and the compact definitions of the tools, keeping it small and fast.
*   **Offloaded Computation:** The LLM's task—mapping language to function calls—is computationally cheap. The expensive work of database queries and data processing is handled by optimized, traditional code.
*   **Enrichment via a Free-Tier Data Ecosystem:** The intelligence isn't just from the LLM; it's from the entire ecosystem of tools. When Spotify deprecated key discovery APIs, the system's capabilities were extended by adding new tools that integrate with the free tiers of Last.fm and MusicBrainz. This allows for rich data enrichment—pulling in community-sourced genre tags, artist bios, and similar-artist information—without relying on a single, fallible, or costly data source.
*   **Deterministic and Reliable:** The core logic of the application resides in code, not in the probabilistic outputs of an LLM. This makes the system's behavior predictable and reliable. The LLM handles the fuzzy language input; the application handles the precise logic.
*   **Extensible and Maintainable:** As the project evolves, we can easily add new capabilities by simply adding new tools. For instance, when certain Spotify API endpoints were deprecated, the project pivoted by adding new tools that pulled richer data from Last.fm and MusicBrainz. The LLM could use these new tools immediately without any changes to the model itself.

## Conclusion

Free-tier LLMs are not just toys or demo tools. When leveraged correctly, they can be the centerpiece of powerful, practical applications. The key is to shift our thinking: instead of asking the LLM to *know* everything, we should empower it to *ask* the right questions. By building a robust set of tools and using the LLM as an intelligent orchestrator, we can create systems that are far more capable than the sum of their parts, all without breaking the bank.
