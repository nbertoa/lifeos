# AI Playbook

## Purpose

This playbook defines how AI assistants should work within LifeOS and how Nicolás should evaluate AI-assisted output. It
applies to ChatGPT, Codex, and other systems regardless of model or interface.

AI-generated content is provisional until a human reviews and accepts it. Generation does not create truth, transfer
engineering responsibility, or authorize publication.

## Appropriate Uses of AI

Use AI to:

- Explore alternatives and make trade-offs explicit.
- Explain unfamiliar concepts at an appropriate level.
- Research with primary and official sources.
- Draft and revise prose.
- Generate boilerplate, scripts, tests, and initial implementations.
- Review code, documentation, architecture, and assumptions.
- Summarize repository context and trace relevant links.
- Automate repetitive, reversible work.
- Identify inconsistencies, missing evidence, and scope growth.
- Help distill durable lessons after a task.

AI is less appropriate when the necessary source is unavailable, a decision depends on private human values not recorded
here, or output cannot receive review proportionate to its risk.

## Repository-First Reasoning

Before answering about Nicolás, his career, working preferences, or LifeOS:

1. Read [LifeOS](../About/LifeOS.md) for system responsibilities.
2. Read [Profile](../About/Profile.md) for concise identity context.
3. Locate the canonical document for the specific fact or decision.
4. Inspect related documents for constraints and potential contradictions.
5. Use external sources only when the repository does not own the fact or current verification is required.

Do not rely on model memory when repository evidence exists. Do not treat a search result, public profile, generated
summary, or derived résumé as more authoritative than the relevant canonical LifeOS source.

## Fact Verification

- Separate repository facts from external facts and inference.
- Verify names, dates, roles, employers, clients, credentials, technologies, and links before publication.
- Preserve distinctions such as employment by Globant and client work for DreamWorks Animation or Sony Interactive
  Entertainment R&D.
- Prefer primary sources and official documentation for technical claims.
- Verify information likely to change, including current roles, credentials, product behavior, and tool interfaces.
- Cite external sources when the claim benefits from traceability.
- Report conflicting evidence rather than silently selecting a claim.

When evidence is insufficient, omit the claim or mark the exact uncertainty. Plausibility is not verification.

## Avoiding Invention

Never invent:

- Personal history, beliefs, preferences, or private thoughts.
- Employers, clients, dates, titles, credentials, achievements, or production credits.
- Project details or responsibilities not present in an approved source.
- Technical behavior, APIs, tests, or validation that were not verified.
- A quotation or personal voice that Nicolás did not provide.

Do not strengthen language during summarization. “Interested in” must not become “expert in”; “worked with a client”
must not become “employed by the client”; a current goal must not become an achieved status.

## Asking for Clarification

Ask when missing information would materially change:

- Factual accuracy.
- Scope or canonical ownership.
- Security, privacy, or confidentiality.
- A destructive or difficult-to-reverse action.
- Publication, commit, push, or external communication.
- The meaning of a personal value or preference.

Otherwise, use a conservative assumption, state it, and continue. Do not ask for facts that can be discovered safely
from the repository or relevant authoritative sources.

## Canonical Facts and Generated Output

Canonical documents own accepted facts and decisions. Generated output is a proposal or derived presentation until
reviewed.

When drafting:

- Link to the canonical source.
- Preserve the source's qualification and uncertainty.
- Keep audience-specific phrasing out of the canonical fact unless it adds durable value.
- Correct canonical sources before updating derived documents.
- Do not commit raw chat transcripts or unreviewed generated text.

If accepted output contains a durable lesson, integrate the smallest useful conclusion into the existing canonical
document. Do not preserve the entire conversation.

## Reviewing Generated Code

Review AI-generated code as critically as human-generated code:

- Confirm it solves the intended problem.
- Inspect the complete diff and relevant surrounding code.
- Verify APIs against current official documentation or source.
- Check ownership, lifecycle, invariants, failure behavior, and security.
- Check Unreal reflection, garbage collection, Blueprint exposure, replication, threads, editor/runtime differences, and
  build configurations where relevant.
- Remove speculative abstractions and unrelated cleanup.
- Run verification proportionate to risk.
- Understand consequential behavior before accepting it.

Never use repeated blind prompting as a substitute for debugging. Each iteration should improve the evidence or
understanding.

## Reviewing Generated Prose

Check generated prose for:

- Unsupported or inflated claims.
- Changed employer/client relationships.
- Contradictory dates, titles, credentials, and technologies.
- Repetition across documents with different responsibilities.
- Generic advice that is not personally adopted.
- False certainty and hidden inference.
- Inconsistent voice or unnecessary marketing language.
- Broken links and stale navigation.
- Disclosure of private or confidential information.

Prefer a smaller accurate document over a complete-looking generic essay.

## Security and Privacy

Do not provide AI systems with secrets, access tokens, passwords, private keys, confidential documents, personal
identifiers that are not needed, or production data without explicit authorization and appropriate controls.

Before using external AI services, consider data retention, training policies, access boundaries, contractual
obligations, and whether the task can be completed with less sensitive context. Redact or minimize data where practical.

Generated commands and code must be reviewed for destructive operations, credential exposure, unsafe network access,
supply-chain risk, and overly broad permissions. Never place secrets in prompts, logs, source control, or generated
examples.

## Employer and Client Confidentiality

- Use only approved, public, or repository-supported career information.
- Preserve the distinction between employer and client.
- Omit proprietary implementation details, internal metrics, unreleased work, code names, infrastructure, source code,
  and private communications.
- Do not infer production credits from tool work.
- When uncertain whether information is public or permitted, omit it and request review.

Factual accuracy does not by itself authorize disclosure.

## Prompting Practices

A useful request gives AI:

- The desired outcome and intended audience.
- Relevant canonical files or repository scope.
- Constraints, exclusions, and authorization boundaries.
- The required evidence or verification level.
- Expected output form.

Ask the assistant to:

- State assumptions and uncertainty.
- Compare alternatives when a decision is open.
- Explain consequential choices.
- Challenge contradictions and unsupported claims.
- Inspect existing conventions before editing.
- Keep changes cohesive and review the final diff.

Avoid prompts that reward confidence, volume, or automatic agreement. Do not ask for a “complete” document when the
available facts are incomplete.

## Suggested ChatGPT Workflow

Use ChatGPT primarily for discussion, learning, research, synthesis, and drafting:

1. Provide the relevant canonical context.
2. Ask it to separate facts, inferences, and questions.
3. Explore alternatives or request a bounded draft.
4. Challenge claims and request sources where needed.
5. Review the result manually.
6. Move only the accepted durable conclusion into LifeOS.

Do not treat a conversation history as a canonical record.

## Suggested Codex Workflow

Use Codex for repository-aware analysis and implementation:

1. Define scope, authorization, and expected result.
2. Require inspection of repository instructions and relevant files.
3. Let it diagnose with read-only checks before editing.
4. Keep implementation focused and preserve unrelated work.
5. Run appropriate validation and inspect the complete final diff.
6. Review facts, design decisions, and remaining uncertainty.
7. Commit or push only when explicitly authorized.

For engineering work, use the [How to Think Like Me](../About/HowToThinkLikeMe.md) guide
and [Engineering Playbook](../Programming/EngineeringPlaybook.md) as working context.

## When AI Output Should Not Be Committed

Do not commit output when:

- It is unreviewed or not understood.
- Factual claims are unsupported or contradictory.
- It contains confidential, personal, licensed, or security-sensitive material.
- It is raw research, brainstorming, a transcript, or temporary task state.
- It duplicates an existing canonical responsibility.
- It documents external material without a durable personal conclusion.
- Generated code lacks verification proportionate to its risk.
- The text is generic and does not improve a future decision.
- Its only purpose is to make the repository appear complete.

“No change” is a valid result of AI-assisted work.

## Recording Durable Lessons

After AI-assisted work:

1. Identify what was actually learned or decided.
2. Confirm it through evidence or explicit adoption.
3. Ask whether it is likely to matter again.
4. Find the existing canonical document that owns it.
5. Record the smallest reusable conclusion and relevant trade-off.
6. Link it only where the relationship improves navigation.
7. Remove temporary prompts, logs, and research artifacts.

Record the lesson, not the entire path used to generate it.

## Long-Term Standard

Use AI to increase understanding, quality, and useful speed without weakening ownership. Begin with repository evidence,
verify consequential claims, protect privacy and confidentiality, review output according to risk, and preserve only
accepted knowledge that remains valuable after the conversation ends.
