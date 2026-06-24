# Architecture Note

## Required Track: Commit-Reveal

The required implementation stores only a commitment during the submission phase:

```solidity
keccak256(abi.encodePacked(answer, salt, msg.sender, bountyId))
```

Including `msg.sender` and `bountyId` binds the commitment to one participant and one bounty, so another participant cannot copy the commitment and reveal it as their own. After the submission deadline, participants reveal their answer and salt. The contract recomputes the hash, stores only valid reveals, and makes unrevealed commitments ineligible for judging or payout.

This design works on any EVM chain and is simple to verify on-chain. Its limitation is that answers become public during the reveal phase, before AI judging is called.

## Advanced Track: Ritual-Native Hidden Submissions

In a Ritual-native design, participants encrypt answers for a Ritual TEE executor or privacy/key flow. The contract stores compact public metadata such as submitter address, encrypted submission reference, ciphertext hash, and status. Plaintext answers exist on the participant device before encryption and inside the TEE-backed judging environment during `judgeAll()`.

The LLM should receive all submissions together in one batch request from the private execution workflow. This allows the AI to compare submissions fairly with the same context. After judging, the system can publish a revealed answer bundle off-chain and store `revealedAnswersRef` plus `revealedAnswersHash` on-chain. The AI recommends a winner, but the bounty owner still finalizes the payout.

## Public vs Hidden Data

Public data:

- Bounty title, rubric, reward, deadlines, and owner.
- Commitment hashes.
- Reveal status.
- AI review bytes or final result reference.
- Winner and payout transaction.

Hidden until reveal or private judging:

- Plaintext answers.
- Salts.
- Encrypted storage credentials.
- Any private inputs used by the Ritual TEE workflow.
