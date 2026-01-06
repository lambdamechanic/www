---
title: tooltest
description: a fuzz checker for your MCPs
author: Mark Wotton
---

# Tooltest: because agents deserve good tools too

A hedgehog is a person with one big idea. A fox is one with hundreds of little ones.

By nature I think I'm a fox, but I have now implemented a fuzz checker for Servant and reimplemented Minithesis twice in new languages. It's probably fair to say that property testing is at least a minor hedgehoggian obsession.

With that in mind, I present the initial release of [tooltest](https://github.com/lambdamechanic/tooltest). This is, roughly speaking, just my reimplementation of [roboservant](https://github.com/mwotton/roboservant), but where Servant is used by a minority of type-obsessed Haskellers, MCPs are everywhere.

Essentially, your MCP publishes a schema of what each tool takes in and promises to put out. It seems simple enough, but frequently MCPs do not actually support what they promise they do, and it can be weirdly hard to detect: your agent will often find workarounds & hacks that make it look like the MCP is working.

Tooltest checks every contract in a much less forgiving manner. Point it at your HTTP or stdio-based MCP server and let it go to town. It will find:

## valid inputs that your server rejects or crashes on

This is table stakes. If you claim I can pass you a string, and you fall in a heap just because I pass you a Turkish dotless ı, your code is broken and you should fix it.

## outputs that don't fit the output schema you specify

You won't see this as much in typed languages, but it's still common in dynamic ones. You could fairly easily do this without Tooltest, but it's a freebie.

And often the way it manifests in actual agent setups is especially obnoxious: your agent process throws a schema validation error from unfamiliar code, and then the agent heroically tries to cover it up with… whatever it can think of.

(Also: if your JSON Schema uses `pattern`, be aware that Tooltest treats those patterns as ECMAScript regexes — which means fun footguns like ASCII-only `\d` / `\w`. That's a whole category of “it worked in my head” bugs.) 

## valid _sequences_ of inputs that are rejected

This is a bit less common in MCP tools than web APIs, mostly because most MCPs are stateless and if a sequence of queries provokes a failure, probably there's a query on its own that provokes it.

However, if you do have some local state and pinky-promise that it is safe to muck around with, Tooltest can try hammering it with sequences of commands to provoke a state-dependent error.

**TODO:** `--clean-slate` is coming (a hook/tooltest-side callback so it can reset state between runs). Until that lands, run Tooltest against a throwaway instance you can reset out-of-band (containers, temp dirs, disposable accounts, etc).

## tools that are unreachable from a base of primitive values you specify

This is where we depart from basic fuzz testing and start to use state-machine techniques.

One of the problems in both web APIs and MCPs is that they're full of sparse IDs like UUIDs. In order to properly explore the full state space of the server, we need to be able to parrot back IDs that we've fetched from the service in the first place.

So Tooltest maintains a little corpus of values mined from responses. In strict mode, required fields are satisfied from that corpus (plus any seed values you provide). This gives you a brutally honest picture of what an agent can reach starting from scratch.

Example: you have a `create_thing` tool that returns an ID, and a `get_thing` tool that requires that ID. Without sourcing/mining, `get_thing` is basically unreachable unless you seed an ID or the system learns it from `create_thing`.

If you want Tooltest to *bootstrap* from the schema when the corpus is empty, pass `--lenient-sourcing` (or set it in `--state-machine-config`). 

# How to use it

The cute part about this is that you mostly don't have to use it at all, or at least not by hand. It's pretty easy to just expose Tooltest to your coding agent, and have the agent fix bugs as it goes. The agent gets full but minimised context (we use shrinking techniques from property-testing), so can usually spot the problem quite quickly.

But it also works as a normal CLI / CI gate:

```bash
# install (prebuilt)
curl -fsSL https://raw.githubusercontent.com/lambdamechanic/tooltest/main/install.sh | bash

# test a stdio MCP server
tooltest stdio --command ./path/to/your-mcp-server
# optional: --arg ..., --env KEY=VALUE, --cwd /somewhere

# test a Streamable HTTP MCP endpoint
tooltest http --url http://127.0.0.1:8080/mcp
# optional: --auth-token "Bearer …"

# make it louder / more repeatable
tooltest --cases 100 --json stdio --command ./path/to/your-mcp-server
Do NOT run this blindly on MCP servers that have destructive tools. If you have a function that deletes your repositories or fires missiles, Tooltest will merrily call it as often as it can to try to provoke a failure.

TODO: --whitelist is coming (so you can explicitly name the safe tools).
TODO: --blacklist is coming (so you can ban the obviously unsafe tools).

Until those land: point Tooltest at a test instance, or temporarily hide/disable destructive tools in your MCP while fuzzing.

If you want the “agent fixes it while I go do something useful” loop, I have a sample prompt here. Set Codex and Claude to fixing your tool's bugs while you do something more interesting.
