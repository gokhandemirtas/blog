---
title: "JeanClawd: Friendly neighbourhood AI"
description: "Reflections about building a local AI assistant"
date: 2026-07-29
tags: [local-ai, ollama, lm-studio, llms, python, vibe-coding, personal-assistant]
draft: false
---

{{< repository-link url="https://github.com/gokhandemirtas/jeanclawd" name="JeanClawd" description="Browse the source code and documentation." >}}

I started JeanClawd because I wanted to find out how far “vibe coding” could take me.

I was also interested in a more practical question: how much of a useful home assistant can
run on hardware I actually own? I like working within constraints, and I have been increasingly
interested in running some smaller models on an ESP32 or Raspberry PI. 

There is something absurd about a network-attached microphone/satellite (e.g. Alexa) sending a voice recording
to a remote data center to be processed, so that my ceiling light can be switched on. I am also uncomfortable
with the amount of energy a data-center use for my trivial asks, and data privacy. So JeanClawd is an
attempt to keep nost of the moving parts local, and allows me to understand the challenges.

While I'm shamefully aware this isn't in any way or shape a breakthrough, it's been a fun an educative experiment
that does a lot of useful things for me. I am a software developer with focus on JS & Typescript professionally,
but I have been clumsily using Python with my pet projects, and I would not have built this nearly as quickly
without Claude and Codex. The code is mine in the sense that I chose the direction, evaluated the results,
and kept changing the design. But AI-assisted coding did a lot of the typing and made my tendency toward
grandiose scope creep considerably worse.

## Scope creep

My personal projects tend to start with a small moon and immediately mutate into a jupiter.

The initial idea was a command line chatbot. Then came questions like:

- How about also a mobile app to access the chatbot when I'm away?
- Yeah I want it to check the weather, Microsoft stocks, news, oh an also set a reminder, and take notes, aaand make plans....
- Oh I definitely want to speak with it, and while I'm at it let it speak back

That is how a terminal chatbot became a Python application with a CLI, REST and WebSocket API,
Telegram interface, MCP server, voice input and output, vision, persistent memory, resumable
sessions, research agents, personas, and external-service connectors.

The name is a play on OpenClaw, and a tribute to Jean-Claude Van Damme, one of my childhood
heroes. JeanClawd is nowhere near as capable as OpenClaw. It may have potential, or it may not.
That uncertainty is part of the fun.

## The hardware constraint is part of the design

I initially worked on my PC running Ubuntu equipped with an RTX 3090 Founders Edition. 
It's great for running a single model, but 24 GB of VRAM still was a hard limit.

I initially used llama.cpp because I wanted a truly portable solution using models in popular GGUF format,
and not to depend on LM Studio or Ollama. Later I gravitated toward LM Studio, partly because it's fast, easy, and has MCP support.
Adjusting model parameters is simple so you can plan the memory usage, and LM Link allows me to offload
inference to other machines on my network.

That was not a complete abandonment of llama.cpp. The provider remains available, can be switched on with a flag.
On my current M1 Mac Studio, MLX support and plethora of MLX models make LM Studio more attractive.

The Mac is slower than the PC for raw inference, but unified memory is a blessing here. It can
keep all five models running in parallel without making me constantly choose which one to unload.
Earlier in the project I was hotswapping models, which proved detrimental as it was always a cold start.

### Performance

Here are the [benchmark](https://github.com/CodeGeekR/benchmark-ai-tops) results from the M1 Mac Studio used for this project:

| System component | Result |
| --- | ---: |
| Operating system | Darwin 25.1.0 |
| Memory | 64.0 GB |
| CPU | ARM · 10 physical / 10 logical cores |
| GPU | Apple MPS · 24 cores |
| NPU | Enabled |

| Benchmark | Result |
| --- | ---: |
| CPU baseline (FP32) | 293.23 GFLOPS |
| GPU Metal (FP16) | 7.06 TOPS |
| NPU (FP16) | 10.72 TOPS |
| NPU (quantized W8A16) | 11.38 TOPS |

| Technical performance report | Result |
| --- | ---: |
| CPU (general processing) | 293.23 GFLOPS |
| GPU (graphics / basic AI) | 7.06 TOPS |
| NPU (high-precision AI) | 10.72 TOPS |
| NPU (quantized W8A16) | 11.38 TOPS |

The approximately 11.4 TOPS result represents roughly 50% of the theoretical peak
reported by this benchmark. This is the maximum measured performance without data
calibration using W8A16. Reaching the maximum TOPS figure with W8A8 requires a real,
trained model for data calibration.

On this hardware, I am getting the following inference performance with LM Studio. These
figures come from a small sample of live requests rather than a controlled benchmark.

| Model | Requests | Server generation | Measured generation | Average TTFT | End-to-end throughput |
| --- | ---: | ---: | ---: | ---: | ---: |
| Qwen3 30B A3B Instruct 2507 (MLX) | 3 | 50.02–58.28 tok/s | 51.49–75.08 tok/s | 2.24–4.99 s | 7.26–14.93 tok/s |
| Meta Llama 3.1 8B Instruct | 1 | 58.85 tok/s | 80.09 tok/s | 7.62 s | 2.79 tok/s |

Across the three Qwen requests, the averages were approximately 55.37 tok/s server-side,
64.96 tok/s as measured by the client, 3.09 seconds time to first token, and 10.26 tok/s
end-to-end. The end-to-end figure includes prompt processing and therefore drops sharply
when the generated response is short.

### Backend comparison

These runs used Qwen3 8B Q4 K-M quantization with a 256-token output limit, resulted in negligible difference in speed.

| Backend | Model | Median latency | Median wall-clock rate | Median decode rate | Output tokens |
| --- | --- | ---: | ---: | ---: | ---: | ---: |
| Ollama | `qwen3-8b-q4km` | 6.767 s | 37.83 tok/s | 38.74 tok/s | 256 |
| LM Studio | `qwen/qwen3-8b` | 6.766 s | 37.84 tok/s | 37.84 tok/s | 256 |
| llama.cpp | `Qwen3-8B-Q4_K_M.gguf` | 6.695 s | 38.24 tok/s | 38.24 tok/s | 256 |

### Relative results

| Metric | Fastest | Difference versus slowest |
| --- | --- | ---: |
| Latency | llama.cpp | 1.05% faster than LM Studio |
| Wall-clock throughput | llama.cpp | 1.06% faster than LM Studio |
| Decode throughput | Ollama | 2.32% faster than LM Studio |


## Task delegation for models

The current model arrangement is intentionally split:

```text
user request
    │
    ├── intent model      → classify primary intent, route to skill
    ├── generalist model  → classify route intent, fire handler, return response
    ├── generalist model  → infuse the conversation with response, reason and respond
    └── vision model      → analyse an uploaded image
    └── research model    → describe images when needed
```

The generalist is Qwen3 30B A3B Instruct in an MLX 4-bit format. The intent model is an 8B
Llama3.1 model. The vision model is Qwen3-VL 4B. There is a dedicated research model that only is used by search / deep research skills with ReAct pattern.
There is also a separate embedding model for vectorization, and memory retrieval.

I experimented with using a single mixture-of-experts model for routing, reasoning, and response generation. A recurring problem was that routing and intent-resolution context could contaminate the model’s conversational context and produce incoherent answers. I therefore separated intent resolution from execution: routing selects the next skill, then its intermediate reasoning is discarded. Only the selected skill’s result and the normal conversation context continue downstream.

The generalist has a larger, cleaner conversation context (which is adjustable in LM Studio). It handles conversation, content curation, summaries, topic extraction and anchoring the conversation on what was discussed in previous turns, so I could ask follow up questions.

The vision model runs with a smaller context and has one job: describe what it sees. That description is then injected into the wider conversation by the generalist.


## Skills vs tools

Earlier I relied on model tool calling. This failed spectacularly.

The model hallucinated tools, didn't call them half the time or called them with missing arguments.
And not every LLM has tool support. I needed something more reliable, and also I wanted structured outputs that follows my DB schemas.
I initially used [Outlines](https://dev.to/shrsv/taming-llms-how-to-get-structured-output-every-time-even-for-big-responses-445c). It constrains token selection during generation (to validate the JSON schema, therefore it's slow), 
Then I switched to Pydantic which validates post generation, retry if needed.

So when I request: "Create a reminder to bake a cake next Tuesday", the structured output is:
```
{
  "type": "REMINDER",
  "datetime": "2026-08-04T00:00:00",
  "description": "bake a cake"
}
```

A skill is a self-contained piece of code extending a base skill class. It has an identifier,
description, system prompt, and implementation. The implementation can call Python code, an
API, update the DB, then return a structured or unstructured response.

Skills are exposed through a registry, so the agent knows it's own capabilities.
What happens inside a skill is deliberately open-ended—within reasonable safety boundaries.

## Connectors

A connector handles authenticated access to service such as Spotify or an email provider. A skill describes what JeanClawd
can do; a connector describes how it connects to an external service using the user’s credentials.

The connector registry currently has Spotify and Philips Hue connectors. The
remaining awkward part is dependency management: official SDKs tend to become dependencies of
the main project even when some users may never use that connector. I have not solved that
properly yet.

## Long-term and short-term memory

These are two different mechanisms rather than two names for the same store.

Short-term memory is the working context of the current conversation. Recent user and assistant
messages are held in process so the assistant can resolve follow-up questions and maintain a
conversation thread. When that history approaches its limit, older turns are compressed into a
rolling summary. Both the summary and the completed turns are persisted with the session: this
makes a session resumable without putting every historical message into every prompt. When a
session is resumed, JeanClawd restores the saved summary and a bounded set of recent interactions.

Long-term memory is a small, user-scoped profile of durable facts, such as a name, location,
occupation, interests, likes, dislikes, and relationships. It is not a transcript and it is not
intended to remember every detail of a conversation. Completed turns are saved as interaction
records, then an asynchronous LangMem extraction step examines the user message together with the
assistant response. The extractor is instructed to record only information explicitly stated by
the user, avoid guesses and temporary details, merge duplicates, and replace facts when the user
corrects them. LangMem maintains this compact structured profile in MongoDB, with the configured
local embedding model providing semantic search over its fields.

This separation is important. The session summary and recent interactions preserve conversational
continuity; the profile preserves durable personal context across sessions. Explicit application
settings—persona, voice ID, Spotify device, and news length—are kept in a normal database document
instead of being inferred as memories, because configuration should be deterministic.

I first tried Mem0 with Qdrant and the embedding model. I was not happy with the results: the
smaller extraction model often struggled to distinguish durable facts from trivial conversational
details, so the memory became polluted with unusable entries. I then tried a bespoke system that
ran retrospective fact extraction after inactivity and debounced the work so it would not slow the
main request path. The current LangMem approach is more satisfactory because the schema and
extraction rules keep the profile compact, while extraction still runs in the background.

There is an important difference between memory persistence and memory retrieval. A fact can be
correctly stored in MongoDB and still not appear in an answer. Normal conversation turns do not
load the entire profile: explicit questions about the user's profile load the bounded profile
context, while other turns must first look personal and then pass the query through semantic
retrieval. Retrieved facts are capped and recently repeated facts are temporarily suppressed, so
personalization does not dominate every response. This improves prompt hygiene, but it also means
that a weak query classifier, an overly narrow vector search, or irrelevant wording can hide a
fact that is present in the database. The `/memories` command and MCP memory tools query the stored
profile directly and are useful for distinguishing a storage problem from a retrieval problem.

The current implementation consequently has three persistence layers: interaction records for
the conversation archive, session summaries and recent interactions for resumable short-term
context, and the LangMem profile for durable user facts. The same long-term facts can eventually
influence reminders and plans. A personal assistant should not just remember what I like; it should
be able to use that information when helping me decide what to do next.

## Voice interaction

Whisper with base model made speech-to-text straightforward. It is an excellent open-source library and easy to
give an application a voice input path.

The difficult part is wake-word detection. Without a reliable wake word, the alternative is to
keep processing the room in audio segments and constantly search for intelligible speech.
Microphone sensitivity, silence detection, chopping audio into useful segments, and deciding
when to trigger the STT pipeline all took many attempts.

I have not yet tested a network microphone listening to the room. I have identified a couple of
ESP32 boards for that future project.

Text-to-speech also runs locally, but outside LM Studio. The project moved from Pocket TTS to
Supertonic so I could experiment with more expressive voice generation. Both are fast enough to
run on the CPU, which is useful because audio processing does not have to fight the language
models for resources.

TTS cancellation became necessary because I am developing everything at once and constantly
switching to whichever idea appears next. Barge-in is also a basic expectation now. Gemini,
Copilot, and ChatGPT’s mobile applications let users interrupt spoken responses. Without that,
the interaction is turn based, like wireless-radio communication.

## Personas

Personas are mainly me entertaining myself and testing how much tone and behavious I can tune
without breaking the core functionality. Personas mostly flavor communication rather than changing the actual
capabilities of the system.

They have a greeting style, behavior, character, and default voice ID. Voice IDs can also be
configured independently. 

The problem is that the model can get carried away. Sometimes the character becomes more
important than answering the user’s original question. An adaptive persona based on memories
exists too, but it is not very good yet.

The alien persona is probably the most fun example. I added custom vocabulary, and listening to
its chittery clicky voice still makes me laugh.

## Useful features and failed experiments

Today I use JeanClawd for weather, stock information, Hue lights, and deep research. Research
mode writes its findings to a Markdown file that I can read later.

I've added an introspection mode for it to have read-only access to it's source code, then record it's findings in .md files. Although the results are not super impressive, it can first identify itself as an application instead of just an LLM, and (poorly) explain how itself works.

Face tracking is an experiment in uncanny valley. The idea was to track my face, understand facial
expressions, and adjust the assistant’s response. Not a success so far, and I'm not sure if it's a great idea for my mental health.

Follow-up questions are another weak point. I added a topic lock and release mechanism so the
assistant can follow up on an earlier subject while still allowing me to change topics. This is
basic behavior in ChatGPT, but it has been surprisingly difficult to reproduce reliably.

Sometimes the topic remains locked, or the wrong topic is identified. The result can be absurd:
a phrase such as “the user doesn’t like candy” gets sent as a search query to Tavily or DuckDuckGo.
These failures are useful reminders that every layer of an agent system can amplify a small
classification mistake.

## What I have learned so far

The main lesson is that a personal assistant should best be tasked with things you're too lazy to do. In fact, this blog post is a product of me asking the agent to interview me, about itself then write a markdown file for me to review. How crazy is that ?

#### Lesson 1
Like any software product this also is an intersection of concerns, and understanding where boundaries meet (which kept moving, a lot of times). The rest is orchestration, and cohesive response to any random request. It's vibe-coded, and although reviewed by multiple models it may lack finesse if it was hand coded by a seasoned Python dev. I've treated the development process as an MVP, and as if I'm working for a startup. If I started from scratch, I'd probably utilize PRD's and more granular commits.

#### Lesson 2
With a background in UI engineering I've long learned to expect the unexpected from users. Of course we try and guide, but they can do anything in any order. So I'm testing the application as an average Joe, then identify what parts needs hardening. Sometimes over hardening a system prompt results in an unresponsive agent, or misclassified intent.

#### Lesson 3
AI-assisted coding changes the economics of experimentation. Claude and Codex made it possible for me to explore far more ideas than I would have implemented manually. That does not remove the need for judgment, or good architecture. It makes judgment more important, because it becomes
easy to create ten plausible abstractions before discovering that the original assumption was wrong. LLM's are calibrated to affirm and validate their users, including our misconceptions and poor understanding. This can amplify bad patterns, security issues, and scalability.

## Future plans
I really like upcycling, and I've got a lot of discarded hardware enclosures that I want to utilize. I'm currently in the process of identifying a list of parts, so that I can build network attached microphone/speaker satellites to access the assistant from different rooms in my house. If possible, wakeword detection and speech recording could be offloaded to these as well as extend the abilities with proximity sensors, temperature and humidity readings from different rooms, and even play some music from my Spotify.

Exciting stuff, but I need to get a solder gun first!
