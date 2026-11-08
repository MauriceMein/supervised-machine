# The Supervised Machine: Daml reference sketch

AI agents on Canton that can only act inside a mandate a licensed human signed, and whose work only counts after that human reviews it.

Companion code for the article **"The Supervised Machine"** by Ace Muir (Maurice Meintzer), asterionai.io:
https://asterionai.io/research/supervised-machine/

Synthetic data only. Research sketch, not production code, not legal advice.

## Run

Requires [dpm](https://docs.canton.network/sdks-tools/cli-tools/dpm) (Daml SDK 3.7.1) and a Java 17 runtime.

```
dpm build
dpm test
```

Verified 2026-11-08 on macOS 15.7 (x86_64), Daml SDK 3.7.1, Temurin 17.0.21:

```
Test Summary (supervised-machine): 10 passed
daml/Test.daml: 10 tests passed
```

(10 = the nine tests below plus the shared `setup` script, which Daml counts as a test.)

## The contracts

| Contract | Signed by | Who sees it | What it does |
|---|---|---|---|
| `MandateProposal` | Professional | Client | Offer of a mandate; client accepts |
| `SupervisionMandate` | Professional + client | Agent | Lists allowed actions, fee cap, approved payees, end date |
| `AgentDraft` | Professional + client | Agent | AI output as hashes; usable only after review |
| `ReviewedWork` | Professional + client | Agent + named parties (e.g. court) | Proof a licensed human approved this exact output |
| `FeePayment` | Professional + client | Agent + payee | Fee paid by the agent within the cap |

## What the tests prove

| # | Test | Shows |
|---|---|---|
| 1 | testHappyPath | Agent drafts, lawyer approves, court sees the receipt |
| 2 | testActionOutsideMandate | Agent can't take an unlisted action |
| 3 | testAgentCannotSelfApprove | Agent can't approve its own work |
| 4 | testAgentCannotForgeReview | Agent can't fake a review receipt |
| 5 | testUncheckedCitationsBlocked | No approval without citation check |
| 6 | testFeeLimits | Fees only to approved payees, under the cap |
| 7 | testRevocation | Client can cut the agent off instantly |
| 8 | testExpiry | Mandate ends on its date |
| 9 | testPrivacy | Opposing counsel sees nothing; court sees no drafts |

## Known limits

- The build warns that templates and tests share one package (daml-script dependency). Fine for a sketch; a real deployment splits tests into their own package.
- `citationsChecked` is a self-reported flag. A real build should point to a hashed, independent citation-check result.
- Settlement of `FeePayment` (e.g. a CIP-56 token transfer) is out of scope.

## Disclosure

asterionai.io builds AI systems for businesses and may apply for Canton Foundation funding related to this work.

## License

MIT. See `LICENSE`.
