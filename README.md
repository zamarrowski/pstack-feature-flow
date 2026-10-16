# pstack feature

A [cornercase](https://usecornercase.dev) flow that runs the Feature playbook of [pstack](https://github.com/backnotprop/pstack) with a team of Claude Code agents, one per role, each in its own tab of the same worktree.

pstack is poteto's set of skills and principles for rigorous AI-assisted engineering. Its Feature playbook explores the design on two models, delegates the code, verifies it on the real surface and has two models review it adversarially before the pull request. In a single chat those steps are subagents. Here every step is a real agent with its own model and effort, and an orchestrator that runs the plan and makes the calls.

## Import it

With cornercase 0.12.110 or later:

```sh
cornercase flow import zamarrowski/pstack-feature-flow
```

Or type `zamarrowski/pstack-feature-flow` on the import page of Settings → Agents → flows. With an older cornercase, download `flow.toml` and run `cornercase flow import flow.toml`.

Then start it from an issue (**start a flow**), from the workspaces' `+ New` (**start a flow…**), or from a shell:

```sh
cornercase flow run "pstack feature" --prompt "Add an --until flag to the export command"
```

It needs Claude Code on the machine that runs the cornercase server.

## The plan

| Stage | Roles | Then |
|---|---|---|
| 1 | architect A, architect B, in parallel | The orchestrator scores both designs against a rubric, picks a base and grafts from the other. |
| 2 | coder | Implements the synthesized design in small, ordered commits. |
| 3 | verifier | Back to the coder on failures, at most twice. |
| 4 | reviewer A, reviewer B, in parallel | Back to the coder with the findings to act on, at most three times. |

At the end the orchestrator pushes the branch and opens a pull request with a Conventional Commits title, a short briefing body and the review verdict.

## The roles

The split follows pstack: judgment goes to Opus, code goes to the faster model. Here that is Sonnet, where pstack uses Grok.

| Role | Model | What its prompt carries from pstack |
|---|---|---|
| orchestrator | opus · high | The lead's job. It writes the intent and the rubric, picks the base and the grafts (arena), writes the throughput checkpoint, sorts review findings into act on, consider, noted and dismissed (interrogate), and writes the pull request (opening a PR). It never blocks on the human and attacks the premise when fixes keep failing. |
| architect A | opus · xhigh | The architect runner prompt: grounding first, the caller's usage before the types, a design package (usage, shape, tradeoffs, alternatives, risks), the design principles and the red flags to screen against. |
| architect B | sonnet · xhigh | The same prompt as architect A. The difference comes from the model. |
| coder | sonnet · xhigh | The Feature playbook's rules for the delegate: the design is the contract, laziness protocol, model the domain, build the lever, sequence verifiable units, test behavior, fix root causes, prove it works, no slop, small ordered commits. |
| verifier | sonnet · high | Prove it works on the matching surface, script the check, explain the number. Inconclusive is not a pass. |
| reviewer A | opus · xhigh | The interrogate reviewer prompt, its rubric (correctness, root causes, structure, verification, complexity, security) and its strict code-quality lens. |
| reviewer B | sonnet · xhigh | The same prompt as reviewer A. |

Every prompt is self-contained, so pstack does not need to be installed. Each agent follows the repository's own rules (AGENTS.md, CLAUDE.md, CONTRIBUTING) first.

## Make it yours

- **More diversity.** Opus and Sonnet are one family. For pstack's cross-provider review, give architect B and reviewer B `grok` or `codex`.
- **Hard changes.** pstack sends the hardest changes to its strongest model. Set the coder to `opus` for cross-cutting or subtle work.
- **Cost.** A run starts seven Claude Code sessions, three of them on Opus.

Edit the flow in Settings → Agents → flows after importing it.

## What it leaves out

- `poteto-mode`'s router. This flow is one playbook. Pick it for new or changed behavior.
- The `how` and `why` subagents. The architects ground the design themselves.
- Code bakeoffs with `arena`. A flow shares one worktree, so the architects compete with designs in text, not with code.
- Fresh subagents per round. Members keep their context between rounds.
- Babysitting the pull request after it is opened.

## Trust

Nobody reviewed this flow for you. Its agents run without asking for permission (Claude Code's `--dangerously-skip-permissions`), so read `flow.toml` before you import it. It sets no `env` and no `args`.

## Credits

The prompts are adapted from [pstack](https://github.com/backnotprop/pstack) by Lauren Tan ([poteto](https://x.com/poteto)), first published as [cursor/plugins/pstack](https://github.com/cursor/plugins/tree/main/pstack), under the MIT License. See [LICENSE](LICENSE).
