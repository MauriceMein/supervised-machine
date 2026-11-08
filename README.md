# The Supervised Machine: Daml reference sketch

A local Daml Script sketch of an AI agent working under a mandate that a professional and a client agreed to. The agent can record drafts and fees only within the mandate's terms, and only the professional's party can approve a draft into a review receipt. No AI model, license check, external signing or token transfer is implemented.

Companion code for the article **"The Supervised Machine"** by AceMuir (Maurice Meintzer), asterionai.io:
https://asterionai.io/research/supervised-machine/

Synthetic data only. Research sketch, not production code, not legal advice.

## Run

Install [dpm and a Java 17 JDK](https://docs.canton.network/sdks-tools/cli-tools/dpm), put `dpm` on your PATH and set `JAVA_HOME` to the JDK. Then:

```sh
git clone https://github.com/MauriceMein/supervised-machine.git
cd supervised-machine
dpm install package
dpm build
dpm test
```

Verified 2026-11-08 on macOS 15.7.5 (x86_64), Daml SDK 3.7.1, Temurin 17.0.21:

```
Test Summary (supervised-machine): 20 passed
daml/SecurityTests.daml: 10 tests passed
daml/Test.daml: 10 tests passed
```

`daml/Test.daml` has the nine original tests plus the shared `setup` script, which Daml counts as a test. `daml/SecurityTests.daml` has ten tests added after an independent review.

## The contracts

| Contract | Signatories | Other observers | What it does |
|---|---|---|---|
| `MandateProposal` | Professional | Client | Offer of a mandate; the client accepts |
| `MandateStatus` | Professional + client | Agent | Keeps the end date and approved receipt recipients; archived on revocation |
| `SupervisionMandate` | Professional + client | Agent | Allowed kinds of work, fee cap, approved payees, end date |
| `AgentDraft` | Professional + client | Agent | Output references submitted by the agent; only the professional can approve it |
| `ReviewedWork` | Professional + client | Agent + approved recipients (e.g. the court) | Receipt showing the professional's party approved this draft |
| `FeePayment` | Professional + client | Agent + payee | Fee record within the cap; no funds move |

## What the tests check

Original tests (`daml/Test.daml`):

| # | Test | Checks |
|---|---|---|
| 1 | testHappyPath | Agent drafts, lawyer approves, the court can query the receipt |
| 2 | testActionOutsideMandate | A draft of an unlisted kind is rejected |
| 3 | testAgentCannotSelfApprove | The agent can't exercise the lawyer's Approve |
| 4 | testAgentCannotForgeReview | The agent alone can't create a receipt naming the configured lawyer and client |
| 5 | testUncheckedCitationsBlocked | Approve requires `citationsChecked = True` |
| 6 | testFeeLimits | Fee records only for approved payees and within the cap |
| 7 | testRevocation | After the client revokes, the old mandate rejects new drafts |
| 8 | testExpiry | Drafting fails after ledger time passes the end date |
| 9 | testPrivacy | In this scenario, opposing counsel can't query mandates or drafts, and the court can't query drafts |

Security tests (`daml/SecurityTests.daml`): approval blocked after revocation and after expiry; receipts only to client-approved recipients; no receipt with `citationsChecked = False`, even when created directly; the agent can't take the professional's or client's role; fee boundaries (zero, negative, exact cap, reused mandate, exhausted cap); professional revocation and expired fee records; one-use approval; repeated drafts allowed; a pending draft stays approvable after a normal fee update.

## Known limits

- `citationsChecked`, the hash fields, the model label and the professional's identity are not verified. A ticked box is an attestation, not evidence that citations were checked.
- The hash fields accept any text and nothing is hashed. Never put confidential content in them.
- Signatories authorize creation; they don't necessarily give a fresh cryptographic signature for each contract. The agent acts with authority delegated through the mandate.
- The professional and client together can create a `ReviewedWork` directly, without the Approve step. A verifier has to check trusted party identities, the package and how the receipt was created, not just that it exists.
- Privacy is tested only by queries on one local test ledger, not across separate nodes.
- Fee records don't settle. A CIP-56 token transfer is planned for v2.
- Off-ledger tools (email, e-filing, documents) are outside this model.
- Templates and tests share one package, so the build warns about the `daml-script` dependency. A real deployment splits them.

## Disclosure

asterionai.io builds AI systems for businesses and may apply for Canton Foundation funding related to this work.

## License

MIT. See `LICENSE`.
