---
title: "What Can You Do With a Free-Tier LLM? Quite a Lot, It Turns Out."
author: "DJ Steve Project"
date: "2026-05-27"
tags: ["llm", "architecture", "efficiency", "free-tier", "gemini", "groq"]
---

# What Can You Do With a Free-Tier LLM? Quite a Lot, It Turns Out.

There's a common assumption in the world of AI today: to build something truly powerful and intelligent, you need access to the biggest, most expensive, state-of-the-art Large Language Models. While these frontier models are incredible, they're not always necessary. With the right architecture, even a free-tier LLM can become the brain of a sophisticated, highly personalized application.

Enter "DJ Steve," a project born from a simple desire: to create Spotify playlists based on my *actual* listening history, not just the whims of a generic algorithm. It's a voice-controlled music assistant that understands requests like "play some punk I haven't heard in a while" or "create a playlist of artists similar to NOFX or Screeching Weasel, but only include tracks I haven't listened to in the last year."

![DJ Steve Chat Interface](https://raw.githubusercontent.com/mikecirioli/uber-mcp/main/blog/images/01-chat-interface.png)

This might sound like it requires a massive model that knows my entire music taste. But it doesn't. The secret lies in a simple but powerful architectural pattern: **the LLM is treated as an orchestrator, not a database.**

## The LLM as a Reasoning Engine

In the DJ Steve architecture, the LLM's only job is to do what it does best: understand natural language and reason about the steps needed to fulfill a request. It doesn't store my listening history, it doesn't know what a "dreamy" song sounds like, and it doesn't directly connect to the Spotify API.

Instead, it has access to a set of over 60 well-defined "tools." These tools are functions exposed by a custom API server (an MCP or "Model Context Protocol" server) that encapsulates all the project's business logic. The heavy lifting—querying a local SQLite database of my listening history, performing fuzzy track deduplication, fetching data from external APIs like Last.fm, and controlling Spotify playback—is all handled by this server.

The LLM's role is simply to be the intelligent switchboard operator, translating a fuzzy human request into a precise sequence of tool calls.

## A Concrete Example: The "Rediscovery" Playlist

Let's walk through a real-world request from the system's architecture plan: **"Play artists I haven't heard in 6 months."**

1.  **User Request:** I speak or type the phrase into the web app interface, which includes widgets like a "Mood Machine" slider and a device picker.
2.  **LLM Receives Text:** The request is first sent to the fastest, cheapest model available—in this case, Groq's Llama 3 8B. It receives only the text of the request and a list of available tools.
3.  **Reasoning and Tool Selection:** The LLM understands the *intent*. It doesn't know the answer, but it knows *how to ask for it*. It determines that the `queryArtistsByRecency` tool is the right one for the job and calls it with the appropriate parameters: `queryArtistsByRecency({ notListenedSince: "6 months" })`.
4.  **Application Code Executes:** The MCP server receives the tool call. This triggers a piece of deterministic, efficient TypeScript code that runs a SQL query against the local `listening_events` database. This is fast, cheap, and precise.

    ![Example of the LLM deciding to call a tool](https://raw.githubusercontent.com/mikecirioli/uber-mcp/main/blog/images/03-tool-call-example.png)

5.  **Data Returned to LLM:** The server returns a list of artist names to the LLM.
6.  **Multi-Turn Orchestration:** The LLM now has the data it needs to proceed. It continues the "conversation" by calling other tools in sequence: `getArtistTopTracks()` for the neglected artists, followed by `createPlaylist()` and `addTracksToPlaylist()` to build the final result in Spotify.
7.  **Natural Language Response:** Once the orchestration is complete, the LLM's final job is to compose a user-friendly response, like: "I've created a new playlist called 'Rediscovery Mix' with tracks from artists you haven't listened to in over six months."

    ![The final chat response after tool execution](https://raw.githubusercontent.com/mikecirioli/uber-mcp/main/blog/images/02-playlist-result.png)

The entire process results in a new playlist, created and populated in Spotify, ready to play.

![The final generated playlist in Spotify](https://raw.githubusercontent.com/mikecirioli/uber-mcp/main/blog/images/04-spotify-playlist.png)

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
