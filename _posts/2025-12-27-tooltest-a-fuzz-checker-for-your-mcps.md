---
title: tooltest
description: a fuzz checker for your MCPs
author: Mark Wotton
---
# Tooltest: because agents deserve good tools too

A hedgehog is a person with one big idea. A fox is one with hundreds of little ones.

By nature I think I'm a fox, but I have now implemented a fuzz checker for servant and reimplemented Minithesis twice in new languages. It's probably fair to say that property testing is at least a minor hedgehoggian obsession.

With that in mind, I present the initial release of [tooltest](https://github.com/lambdamechanic/tooltest). This is, roughly speaking, just my reimplementation of [roboservant](https://github.com/mwotton/roboservant), but where Servant is used by a minority of type-obsessed Haskellers, MCPs are everywhere. Essentially, your MCPs publish a schema of what each tool takes in and promises to put out: it seems simple enough, but frequently MCPs do not actually support what they promise they do, and it can be weirdly hard to detect: your agent will frequently find workarounds & hacks that make it look like the MCP is working.

Tooltest checks every contract in a much less forgiving manner. Point it at your HTTP or stdio based MCP server and let it go to town. It will find:

## valid inputs that your server rejects or crashes on

This is table stakes. If you claim I can pass you a string, and you fall in a heap just because I pass you a turkish utf-8 character, your code is broken and you should fix it.

## outputs that don't fit the output schema you specify 

You won't see this as much in typed languages, but it's still common in dynamic ones. You could fairly easily do this without Tooltest, but it's a freebie, and often the way it manifests in actual agent setups is that your agent process throws a schema validation error from unfamiliar code that your agent then heroically tries to cover up.

## valid _sequences_ of inputs that are rejected. 

This is a bit less common in MCP tools than web apis, mostly because most MCPs are stateless and if a sequence of queries provokes a failure, probably there's a query on its own that provokes it.

However, if you do have some local state and pinky-promise that it is safe to muck around with, Tooltest can try hammering it with sequences of commands to try to provoke a state-dependent error. It is your responsibility to pass a function to "--clean-slate" so tooltest can reset in between runs.

Do NOT run this blindly on MCP servers that have destructive tools. If you have a function that deletes your repositories or fires missiles, tooltest will merrily call it as often as it can to try to provoke a failure. You can pass a whitelist or a blacklist to `tooltest` to make sure destructive functions don't get called.
 
## tools that are unreachable from a base of primitive values you specify

This third one is where we depart from basic fuzz testing and start to use state-machine techniques. One of the problems in both web APIs and MCPs is that they're full of sparse ids like UUIDs: in order to properly explore the full state space of the server, we need to be able to parrot back ids that we've fetched from the service in the first place. If you don't pass "--lenient-generation", tooltest will only use what you provide as inputs, and you'll get a good idea of how accessible your tool is to agents starting from scratch.


# How to use it

The cute part about this is that you mostly don't have to use it at all, or at least not by hand. it's pretty easy to just expose tooltest to your coding agent, and have the agent fix bugs as it goes. The agent gets full but minimised context (we use shrinking techniques from property-testing), so can usually spot the problem quite quickly: I have a sample prompt [here](https://github.com/lambdamechanic/tooltest/?tab=readme-ov-file#agent-assisted-fix-loop-prompt). Set codex and claude to fixing your tool's bugs while you do something more interesting.


