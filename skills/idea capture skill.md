idea capture skill

<skill_summary>
Records ideas suggested by the user as Idea items in Innovator. Saves the user's words verbatim in
`exact_input`, writes a brief title and a description, sets `ideator` to the user's email from
WhoAmI, and creates one Idea per distinct idea. Tells the user that their account will be
associated with the idea before anything is saved. DRAFT: the create tool name and input format
are placeholders to confirm against the agent's real toolset.
</skill_summary>

## When to Call
- "I have an idea: ..." / "Suggestion: ..." / "What if we ..."
- "Can you log this idea?" / "Record these ideas for me"
- Any message where the user proposes an improvement, new feature, product idea or process change
  and wants it recorded.
- Defects, complaints and problems the user asks to record as an idea. Record them as Ideas like
  anything else. Do not reroute them to another skill or ask whether they belong somewhere else.

Do NOT use for:
- finding or listing existing ideas -> "Edge API Usage".
- editing or deleting an existing Idea. This skill only creates new ones.

If the user only mentions an idea in passing and has not asked to record it, offer to record it
first.

## Critical Limitations
<important>
1. `exact_input` holds the user's words EXACTLY as typed: no spelling fixes, no trimming, no
   reformatting, no summarizing. Copy it character for character.
2. NEVER send `id`, `item_number` or `keyed_name`. The server assigns them.
3. Before saving, ALWAYS tell the user that their user account (their email) will be associated
   with the idea, and wait for them to agree (W4). Never save first and inform afterwards.
4. Get the email from `who_am_i` in this conversation. Never infer it from earlier context, and
   never type an email the user gave for someone else into `ideator`.
5. The title and description must only restate what the user said. Do not add features,
   benefits, numbers or reasoning the user did not give.
</important>

## Fields
| Idea property | Value | Source |
|---------------|-------|--------|
| `exact_input` | the user's message, verbatim | user (W2) |
| `title` | brief title, at most about 60 characters | agent (W3) |
| `description` | 1 to 3 sentences restating the idea clearly | agent (W3) |
| `ideator` | the user's email | `who_am_i` (W1) |
| `item_number` | idea number | server. Never send |

## Building Blocks

W1 Identify the user. Call `who_am_i` and read `email`. If `email` is empty, tell the user that
their account has no email address, and ask whether to use their login name (`login_name`)
instead or not to save. Do not guess an email.

W2 Find the ideas and the exact input.
- `exact_input` is the user's message that contains the idea. If the idea was given across more
  than one message (for example, the user added a detail after a question), store those messages
  verbatim, in order, separated by a blank line.
- Split into separate ideas only when the user proposes things that could each be done on their
  own ("add dark mode, and also let us export to PDF" is two ideas). Several details about one
  proposal are one idea.
- If it is unclear whether something is one idea or two, ask once, then go with the answer.
- With several ideas, every Idea item gets the full original message in `exact_input`, so the
  context is kept. The title and description cover only that item's idea.
- Do not ask follow-up questions to improve the idea. Record what the user said. If the message is
  too vague to write a title (for example, "I have an idea"), ask what the idea is.

W3 Write the title and description.
- Title: short and specific, naming the change being proposed, or the problem if the user
  described a defect without proposing a fix. Examples: "Dark mode for the portal", "Export BOM to
  PDF", "Printer jams mid-print". No "Idea:" prefix, no user names, no dates.
- Description: 1 to 3 sentences in neutral third person, stating what is proposed and, if the user
  said so, why. Keep the user's terms.

W4 Inform and confirm. Show what will be saved, then the account notice, in one message:

**Idea 1:** {title}
{description}

**Idea 2:** {title}
{description}

"Saving will associate these ideas with your user account ({email}). Your original message will
be stored word for word with each idea. Should I save them?"

Apply any edits the user asks for and show the list again. Save only after a clear yes. If the
user declines, save nothing.

W5 Create. For each idea, call the create tool (placeholder: confirm the real tool name and input
format) on entity set `Idea`:
{"title":"...","description":"...","exact_input":"...","ideator":"{email}"}
One call per idea. If one fails, still create the others, then report which failed and why. Do not
retry a failed idea with changed fields unless the user agrees.

W6 Report back. List every created Idea with the `item_number` and title exactly as returned by
the server, linked per "Innovator Integration". Never state a number you did not receive.

## Rules
- One Idea item per distinct idea. Never merge two ideas into one item, and never split one idea
  into several.
- `exact_input` is never edited, not even to remove a typo, unless the user asks.
- If the message contains passwords, credentials, or personal or customer data, point it out
  before saving. `exact_input` stores it verbatim, so offer the user the chance to reword the
  message first.
- Never save without the account notice and the user's yes.
