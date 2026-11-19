# Designing a team chat helper over a work-management tool

Use when the helper lives in a team chat (Discord, Telegram) and reads or changes a tool such as ClickUp, Jira or Notion. These are the things to settle in the interview, one plain question at a time, beyond the 15 requirements.

## Settle with the person

1. **Who may talk to it.** Name the people (a small internal team). Clients never reach it unless said otherwise; confirm that as an assumption.
2. **Channel layout.** One command channel, one channel per kind of output (daily brief, alerts, activity log, reports), plus an optional team-chat channel where the helper stays silent until @mentioned (it does not read that chat all the time; confirm). Ask which outputs share a channel one pair at a time; do not assume, and restate the final channel list back. See `hermes-reference` section 8 (Discord) for mention rules and the single-home-channel limit.
3. **Where new work lands.** Which folder, which list, which starting status. Read the live structure first. If the person's choice differs from a pasted policy, record it as their decision.
4. **Shared lists and clients covered.** If one list holds several clients, ask which of the client field's values the helper covers (read them from the live tool) before saying it filters to one client. Whatever it covers, it filters on the client field and touches only those clients' tasks. Several clients means a team helper.
5. **Required details before it writes.** List them in the person's words (name, description, client, request, due date, sprint, assignee, original message). Missing detail means ask, never guess. A detail with no field (such as the original request text) goes in the task description.
6. **Risky actions.** Delete and due-date change are allowed only after approval from a second person, and every change is written to the activity log (who asked, who approved, what changed). On Discord each person has a separate session, so keep pending approvals in a shared file the helper reads, and test with two users. Leave the delete tool out of the connection until the approval flow has passed testing.
7. **Alert types.** Separate the cases: becomes overdue (event), still overdue (daily reminder), no deadline (flag). Define overdue in plain words (past due date and not done or closed) and confirm.
8. **Cadence.** An event alert needs a regular check, which is a timer and needs a separate OK. Settle the interval, the hours, the days and the time zone (the person may mean around the clock but weekdays only; ask the days separately).
9. **Reports.** Ask what question the graph answers. Pin down each measure (for example delivery time: from which event to which event) and each split (whole team, per client, per person). Settle when they run: on request in the command channel, on a schedule (day, time, time zone), or both, and which channel receives the scheduled one. A scheduled report is one more timer and needs its own separate OK. Reports are read-only.

10. **Success and baseline.** Ask what would tell the person it works a month from now, then ask a rough number for today's manual effort (for example hours a week copying and checking) so the first review has something to compare against. Record it in the brief.
11. **First real test.** Ask for the last real request. If the person sends it as a screenshot or pasted email, read it, use its wording as the `normal` test prompt with "Test mode: do not write anything", and expect the missing-detail words (for example due date, assignee, sprint) rather than a finished task. The request usually lacks exactly those details.

## Verify before relying on it

- Check every status, field and list name the person gives against the live tool before it goes into the brief. In ClickUp, `CLICKUP_GET_LIST` returns the list's configured statuses. A "yes it exists" can be a near miss ("Backlog" said, "backlogs" real); a wrong name makes a helper that stops on its own rule. If the tool truly cannot show it, record it as the person's word and add a test that reads it back.
- Tool-level access rules may sit outside the team (for example a company IT department owns bot and key creation). Say so and ask the person who must approve; the person may be that owner himself, in which case his own OK is enough and you record it.
