problem report skill

<skill_summary>
Turns a user's description of an issue into a filed Problem Report (PR). Reads the complaint,
works out what is missing, asks targeted follow-up questions (impact, behavior, steps,
environment), drafts a title and the report sections, confirms the draft with the user, then
creates the PR. The server assigns the PR number and status. DRAFT: the create tool name and the
reported_by binding are placeholders to confirm against the agent's real toolset.
</skill_summary>

## When to Call
- "Something is wrong with ..." / "I want to report a problem" / "log an issue"
- "The printer keeps jamming" / "I get an error when I log in" / "the extruder overheats"
- Any description of a defect, failure, malfunction or error the user wants recorded.

Do NOT use for:
- finding or listing existing PRs, or their status -> "Edge API Usage".
- which changes reference an item -> "Change Management Queries".
- updating, promoting or closing an existing PR. This skill only creates new PRs.

If the user only describes a problem without asking for a report, offer to file one first.

## Critical Limitations
<important>
1. NEVER send server-assigned fields: `item_number`, `state`, `id`, `config_id`, `generation`,
   `major_rev`, `keyed_name`, `created_on`, `modified_on`, `created_by_id`, `modified_by_id`.
2. NEVER create the PR before the user confirms the full draft (W5). A PR is visible to others
   and starts a process.
3. NEVER invent facts. If the user cannot answer, write "Not provided" and say so in the draft.
4. Immediate safety risk (fire, smoke, electrical hazard, injury risk): before any questions, tell
   the user to make the situation safe and follow their site's safety procedure. Filing a PR is
   not an emergency response. Then continue, with severity 1.
5. Allowed `severity` values: "1", "2", "3", "4" (strings). Any other value fails.
6. Leave triage fields empty: `priority`, `classification`, `phase_found`, `phase_caused`,
   `basis`, `verification`, `action`. Fill one only if the user explicitly gives it, and only
   with a value from the schema's allowed list.
</important>

## Fields
| Report section | PR property | Source |
|----------------|-------------|--------|
| PR Number | `item_number` | server. Never send |
| Status | `state` | server. Never send |
| Title | `title` | agent writes it (W4) |
| Application Environment | `environment` | user, via W2 |
| Sequence of Events | `events` | user, via W2 |
| Description | `description` | agent writes it from the user's words (W4) |
| Ramifications | `impact` | user, via W2 |
| Severity | `severity` | agent assesses it with the user (W3) |
| Affected item (optional) | `affected_item` | context item, if a Part, Document or CAD (W6) |
| Reported by (optional) | `reported_by` | current user (W6) |

## Building Blocks

W1 Read the complaint. Before asking anything, sort what the user said into the four sections and
the severity questions. Decide whether the issue is physical (equipment, parts, materials,
workspace) or software (application, website, data). Ask only about what is missing or unclear.

W2 Ask follow-up questions, ONE per message.
- Ask exactly one question, then wait. No headings, no lists of questions, no second question
  tucked into the same sentence ("and is there...").
- At most 5 questions in total. After the 5th answer, or once every section is covered, go
  straight to the draft (W5) and mark gaps "Not provided".
- Skip anything already answered, and never repeat ground already covered (for example, "is there
  a workaround?" after the impact answer already described one).
- If the user says "just file it", draft right away.

Ask in this order, skipping answered steps:

Q1 Impact (feeds Ramifications and Severity). One question covering how much the problem stops
the user, any workaround, and any safety risk. Example: "How is this affecting your work right
now? For example, is printing stopped entirely, or is there a way around it?" If severity is still
unclear, Q5 may follow up on it.

Q2 Expected vs actual behavior (feeds Description). Example: "What did you expect to happen, and
what happens instead?" Ask for exact error text, codes, lights, sounds or smells if not given.

Q3 Steps to reproduce (feeds Sequence of Events). Example: "What steps lead up to the jam, from
starting the print to the point where it fails?" Write the answer as numbered steps ending with
what went wrong. If frequency is unclear, fold it in: "...and does it happen on every print?"

Q4 Environment (feeds Application Environment). Ask about the conditions most likely to matter,
including factors the user may not have connected to the problem. Pick from:
- Physical: machine or model, location, temperature, humidity, dust, vibration, power,
  materials or consumables, recent maintenance, run time.
- Software: application and version, browser, operating system, device, network (VPN, office,
  home), user role or permissions, recent updates or configuration changes, other users affected.
Always include "did anything change just before this started?" Example: "What printer model and
filament are you using, and did anything change, such as new filament or maintenance, before the
jams started?"

Q5 Optional. Only for the biggest remaining gap: unclear severity, a missing step, or a missing
environment factor. Otherwise draft.

W3 Assess severity from Q1 (and Q5 if used). Consider:
- Can the user still do their job? Is work stopped for them or others?
- Is there risk of fire, injury, permanent damage or data loss?
- Is there a workaround? Is it easy, or difficult and time consuming?
- How soon does this need to be fixed?

Apply the first rule that fits:
| Severity | Rule |
|----------|------|
| 1 | Production down: the user cannot do their job, OR there is an immediate safety risk (fire, irreparable damage, data loss), AND there is no workaround |
| 2 | There is a workaround, but it is difficult, OR the problem does not need to be solved immediately |
| 3 | The problem needs to be solved eventually |
| 4 | Nice to have |

An immediate safety risk is severity 1 even with a workaround. If answers conflict, pick the more
severe level. Show the level with a one-line reason in the draft so the user can correct it.

W4 Write the title and description.
- Title: one line, about 80 characters at most, naming what is failing and the symptom, plus the
  key condition if any. Examples: "Extruder head overheats and damages filament", "Cannot log in
  to Print Manager on Firefox". No severity, dates or names. Do not start with "Issue with" or
  "Problem:".
- Description: 1 to 2 plain sentences summarizing the problem. Do not repeat steps or environment.

W5 Confirm, then create. Show the draft:

**Title:** ...
**Severity:** {n}, {one-line reason}
**Application Environment:** ...
**Sequence of Events:**
1. ...
**Description:** ...
**Ramifications:** ...
**Affected item:** {item_number}, or "none"

Then ask "Should I file this problem report?" Apply any edits and show the changed draft again.
Create only after a clear yes. Use the create tool (placeholder: confirm the real tool name and
input format) on entity set `PR`:
{"title":"...","environment":"...","events":"...","description":"...","impact":"...","severity":"2"}
plus the optional bindings from W6.

W6 Optional links.
- Affected item: if the `<current_item>` block (see "Context Item Awareness") is a Part, Document
  or CAD, ask whether the problem concerns it. If yes, bind
  `"affected_item@odata.bind":"Change_Controlled_Item('{id}')"`. Never link an unconfirmed item.
- Reported by: the server may set this. If not, call `who_am_i` and bind the user's alias
  identity: `"reported_by@odata.bind":"Identity('{alias id}')"`. If the user has more than one
  alias identity, leave it empty rather than guess.
- Attachments (photos, logs, screenshots) via `PR_File` are not handled here. If the user mentions
  one, tell them to attach it to the PR in Innovator after it is created.

W7 After the create call:
- Success: report the PR number and status exactly as returned, and link the PR (see "Innovator
  Integration"). Never state a number you did not receive.
- Failure: explain the error plainly, keep the draft, and do not retry with changed fields unless
  the user agrees.

## Writing the sections
- Use the user's own terms, part names, error text and numbers. Do not reword error messages.
- Write in neutral third person ("The printer jams when ..."), not "I" or "you".
- Stay factual. Put cause guesses in Ramifications only if the user offered them, labelled "User
  suspects: ...".
- Leave out personal data not needed to fix the problem (phone numbers, addresses, customer
  names). Never put passwords or credentials in a PR, even if they appear in an error description.

## Rules
- One PR per problem. If the user describes two unrelated problems, offer to file two reports.
- One question per message, at most 5, in the order Impact, Expected vs actual, Steps,
  Environment, then one optional gap-filler.
- Never ask what the conversation already answered, or two questions covering the same thing.
- Never file without explicit confirmation of the final draft.
- Never send server-assigned fields.
