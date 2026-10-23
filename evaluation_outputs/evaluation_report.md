# Deliverable 1 - Evaluation Report

Reference date: 2026-10-20. LLM: `google/gemma-2-2b-it`. Embeddings: `paraphrase-multilingual-MiniLM-L12-v2` + TF-IDF (RRF).

## Metric
The score is **phrase-based**: a question passes when every `must_include` phrase (any listed alternative) appears in the answer and no `must_not_include` phrase appears. It does not measure whether the answer is truly correct.

## Single-turn results (52 questions)
| Configuration | Passed | Rate |
|---|---|---|
| Prompt-only | 6/52 | 11.5% |
| RAG + tools (this rerun) | 21/52 | 40.4% |

Outcome of the per-question comparison: Failed both: 31, Improved: 15, Passed both: 6.

Automatic triage of the remaining RAG failures (preliminary): incorrect_or_incomplete_answer: 19, gold_evidence_missing: 4, retrieval_failure: 4, product_lookup_failure: 4.

## Root causes fixed
1. Spaced order numbers (e.g. '8978 2837') and Devanagari digits were never recognised, so verification could not start; digit groups are now parsed as standalone numbers and Unicode digits are normalised.
2. Verification used ALL digits in a message concatenated together (dates/years/quantities could be mistaken for codes); now only standalone 4/6-digit groups are accepted, dates are excluded, and an order locks after 2 failed attempts (4 per conversation).
3. The model prompt included delivery pincode / phone / customer_id from verified orders; these fields are now removed, and verified order data is kept for later turns.
4. Retrieved context for the CURRENT turn was attached to the FIRST turn of the history in multi-turn chats; each turn now carries its own evidence and history stays clean.
5. English-only embeddings (all-MiniLM-L6-v2) retrieved only FAQH chunks for Hindi/Hinglish queries; now a multilingual embedding model + TF-IDF hybrid (reciprocal-rank fusion) + cross-lingual query hints is used. Follow-up questions reuse the previous question for retrieval.
6. Source precedence was left to similarity; now superseded / not-yet-effective documents are excluded and results are re-weighted by explicit authority (policy > agent guide > help/FAQ > marketing > logs).
7. Product questions relied on policy embeddings; products.csv is now looked up by SKU or product name and the row (and its category terms) is added to the prompt and retrieval query.
8. Refund/return date questions: relative dates are resolved against the 20 Oct 2026 reference date and a code-computed business-day (Mon-Sat, minus holidays) / calendar-day table is supplied instead of asking a 2B model to do date arithmetic.
9. Generic greeting / 'checking, please wait' / unfilled placeholder / leaked section-ID replies are detected, regenerated once with a stricter prompt, then replaced by a safe fallback.
10. Escalation relied on a few keywords and appended a ticket sentence regardless; tickets are now created only for defined escalation reasons, the reply quotes the ID only when create_ticket returns created=True, and model-invented ticket claims are removed.

Unit tests: 30/30 passed.

## Conversation replay
96 turns across 23 conversations were replayed. Automatic red-flag counts (not proof of passing): generic_greeting: 0, stall_phrase: 0, placeholder: 0, ticket_claim_without_real_ticket: 0, escalation_expected_but_no_ticket_tool_call: 1, lookup_expected_but_not_called: 0.

Manual conversation review for this rerun has NOT been completed; no claim is made that all turns pass.

## Remaining limitations
- The pass rate is PHRASE-BASED (must_include / must_not_include substring checks). It is not a human judgement of correctness; semantically correct answers can fail and vice versa.
- A 2B local model (gemma-2-2b-it) can still misread evidence, miscompute, or answer in the wrong language; Hindi/Hinglish quality is weaker than English.
- Holiday dates are auto-detected from the dataset files; the printed list must be checked by a person against the KB calendar.
- Day-counting convention for business days (day 1 = first business day after the event) is assumed unless the policy text says otherwise.
- Orders delivered before a policy version's effective date may need the older version; superseded documents are currently excluded, so such cases need manual review.
- Escalation rules cover human requests, repeat contacts and damage-approval requests only; other cases that policy says need human review rely on the model's answer and create no ticket.
- Tool calls in the replay are recorded from this code's own logs; production logs were not available.
- Manual reviews (failed cases and conversation turns) must be re-done for THIS rerun; reviews of the earlier run do not apply to new outputs.

No API keys, tokens or `.env` files are included in the deliverable.