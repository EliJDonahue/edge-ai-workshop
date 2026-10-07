# Edge AI Workshop: Build a Support Assistant Agent

Resources for the hands-on **Aras InnovatorEdge AI** workshop at the Aras Connect Tech Summits 2026.

## Intro

This repo is for workshop attendees: Aras Innovator administrators, developers, and solution architects who want hands-on time building AI agents on top of their PLM data. You don't need prior AI or agent-building experience. Knowing your way around Aras Innovator helps.

By the end of the session you will have built, published, and used a working agent in your sandbox environment. More importantly, you'll understand how the parts fit together: APIs, access points, skills, and agents. With that, you can go home and build agents for your own use cases.

**Before the workshop:**

- Bring a laptop with a current version of Chrome or Edge.
- Have your Aras CIAM credentials ready. You'll use them to sign in to Edge.
- You'll get sandbox environment credentials at the start of the session. Nothing needs to be installed beforehand.

## What We're Building

We're building a **Support Assistant**: a conversational agent that helps users report problems they find with parts, documents, or other items in Aras Innovator.

Users describe a problem in plain language. The assistant asks follow-up questions to collect the details it needs, confirms them with the user, and then files a **Problem Report** in Innovator for them.

To see where we're headed, try the finished [Support Assistant](https://edge.aras.cloud/ai/chat/0f4cfdaba9894bfe98d945bf) that we'll build together.

The agent is built from these pieces:

| Piece | What it does in this build |
| --- | --- |
| **API** | Exposes the Innovator data and actions the agent needs, such as looking up items and creating Problem Reports. All agent data access goes through the Edge API layer. The agent never touches the database directly. |
| **Audience** | Controls who and what can consume the API. |
| **Access point** | The governed entry point the agent uses to reach the API. |
| **Skill** | A reusable set of instructions for one job, here *filing a problem report*: which fields to collect, how to validate them, and when to ask for confirmation. |
| **Agent** | The Support Assistant itself. It combines a persona and instructions with the skill, tools, and access point. |

Every query runs under the requesting user's own Innovator session and permissions. The agent can only see and do what that user could.

## Flow

Work through these steps in order. Each step builds on the one before it.

You'll work in two places:

- **[Edge API Management Service (AMS)](https://edge.aras.cloud/ams)** for steps 1–2, where you create and configure APIs.
- **[Edge AI platform](https://edge.aras.cloud/ai/platform/home)** for steps 3–9, where you find and configure everything for Edge AI.

### 1. Create an API

In [AMS](https://edge.aras.cloud/ams), create an API that exposes the Innovator operations the Support Assistant needs:

- Search and read items (Parts, Documents) so the agent can identify what the problem is about.
- Create a Problem Report and link it to the affected item.

Keep the API focused. Expose only what the agent needs for this job.

### 2. Set the audience

In [AMS](https://edge.aras.cloud/ams), add this audience to the API so it can be consumed by agents:

```
b832fca416424121bc28a7d3a3a832f2
```

If you skip this step, the API won't be available when you try to connect it to an access point.

### 3. Create an access point

In [Access Points](https://edge.aras.cloud/ai/platform/access-points), create an access point for the API. This is the governed connection the agent will use. Make a note of its name, because you'll select it in step 6.

### 4. Create a skill

In [Skills](https://edge.aras.cloud/ai/platform/skills), create a skill called **File Problem Report**. In the skill instructions, describe:

- **When to use it.** The user reports a defect, issue, or unexpected behavior with an item.
- **What to collect.** The affected item, a title, a description of the problem, and severity. Add any other fields your Problem Report requires.
- **How to behave.** Ask for missing details one at a time, confirm the summary with the user before submitting, and return the new Problem Report number when done.

Write the instructions the way you'd brief a new colleague: specific, in plain language, with examples of good input.

### 5. Create an agent

In the [Agent Library](https://edge.aras.cloud/ai/platform/agents), create an agent called **Support Assistant**. Give it:

- A short description of its purpose, used to route users to it.
- Instructions covering its persona and scope. For example: it's helpful and concise, it handles problem reporting, and it says so when a request is outside its scope.

### 6. Add the skill, tools, and access point to the agent

Open your agent in the [Agent Library](https://edge.aras.cloud/ai/platform/agents) and connect everything you built:

- Add the **File Problem Report** skill. To compare against your own, see the [example problem report skill](skills/problem%20report%20skill.md).
- Add the tools the skill needs, such as item search and Problem Report creation.
- Select the **access point** from step 3.

### 7. Publish the agent

Publish the agent. A draft agent can't be used in chat until it's published.

### 8. Preview the agent

Use preview to test the agent before you rely on it. Try:

- A complete request: *"Part 1234-A has a cracked housing. It's a high-severity issue."*
- A vague request: *"Something's wrong with a bracket."* Does it ask good follow-up questions?
- An out-of-scope request: *"What's the weather?"* Does it stay in its lane?

If the behavior is off, adjust the skill or agent instructions, republish, and preview again.

### 9. Use the agent in chat

Open the Aras AI Assistant, select **Support Assistant**, and file a real Problem Report in your sandbox. Then open Innovator and confirm the Problem Report was created with the details you provided.

## Tips

- **Iterate on instructions, not just configuration.** Most agent behavior problems are instruction problems. If the agent skips a field or submits too early, say exactly what you want in the skill.
- **Let AI draft skills for you.** Show Claude or your favorite AI agent a few existing skills, then ask it to draft a new one for your use case and API. Review and edit the draft before you use it.
- **Say what you want, and what you don't want.** Counterexamples help. For example: "Don't file a report until the user confirms the summary" or "Don't guess the item number if the user hasn't given one."
- **Keep skills narrow.** One skill, one job. A focused skill is easier to test, reuse across agents, and debug.
- **Expose the minimum.** Give the API and the agent only the operations they need. This follows least-privilege practice, and fewer tools also means fewer chances for the agent to pick the wrong one.
- **Always confirm before writing.** For any skill that creates or changes data, have the agent read back what it's about to do and wait for a yes.
- **Republish after changes.** Edits to a published agent don't take effect in chat until you publish again.
- **Test the unhappy paths.** Missing info, ambiguous item names, and requests the user doesn't have permission for tell you more than the happy path does.
- **Run agents where your users work.** Published agents can run in the AI Assistant sidebar inside Innovator or in the standalone chat interface.
- **Stuck?** Raise your hand. Workshop staff are in the room to help.

## Resources

- [InnovatorEdge AI Administrator Guide](https://docs.aras.com/aras-innovatoredge-ai-book/innovatoredge-ai-administrator-guide): Edge AI product documentation
- [Create Your API](https://docs.aras.com/innovator-edge/product-guides/create-your-api): Edge API documentation
- [Ideator agent](https://edge.aras.cloud/ai/chat/24712fc1d95749c595e5e7fe): chat with an agent to share your ideas and feedback
- [Aras Community](https://community.aras.com/) for questions, discussion, and sharing what you build
