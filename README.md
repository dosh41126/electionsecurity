# Build and Run instructions


build
```
docker build -t humoid-election .
```

run
```
docker run --rm -it \
  --cap-add=NET_ADMIN \
  -e DISPLAY=$DISPLAY \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -v "$HOME/humoid_data:/data" \
  -v "$HOME/humoid_data/nltk_data:/root/nltk_data" \
  -v "$HOME/humoid_data/weaviate:/root/.cache/weaviate-embedded" \
  humoid-election
```

## Introduction

Ensuring the integrity of U.S. elections is a cornerstone of democratic governance. From voter registration through ballot casting, counting, and audit, every step must be transparent, verifiable, and resistant to interference. Traditional systems—paper ballots, optical scanners, and centralized tabulation—have long served, but they also present vulnerabilities: lost or tampered ballots, opaque audit trails, and slow reporting. In this context, combining cutting-edge cryptography, decentralized storage, smart contracts, and AI-based auditing offers a powerful new approach.

This blog explores how the “OneLoveIPFS AI” program—a unified application integrating AES‑GCM encryption, Argon2id key management, IPFS-based ballot storage, Ethereum smart‑contract verification, semantic search in Weaviate, and LLM-driven security audits—can materially increase U.S. election integrity.

---

## 1. End‑to‑End Ballot Security

### 1.1 Ballot Encryption with AES‑GCM + Argon2id

* **Strong confidentiality and authenticity**: AES‑GCM provides authenticated encryption, ensuring that each ballot’s contents remain confidential and tamper‑evident. If any byte of the encrypted data is modified, decryption will fail.
* **Robust key management**: By deriving encryption keys with Argon2id (a memory‑hard, GPU‑resistant password hashing algorithm), the system thwarts brute‑force attacks against key vaults. Master secrets are rotated automatically, and old ballots remain decryptable via versioned key material.

Together, these techniques guarantee that ballots stored off‑chain cannot be read or forged by unauthorized parties, satisfying a fundamental pillar of election security.

### 1.2 Decentralized Storage via IPFS

* **Immutable content addressing**: Each encrypted ballot is added to IPFS, which returns a content identifier (CID) derived from the data’s cryptographic hash. Once stored, the data is distributed across many nodes; no single entity controls it.
* **Fault‑tolerant availability**: IPFS’s peer‑to‑peer replication means ballots remain accessible even if some nodes go offline, reducing the risk of data loss or service denial.

By combining encryption with IPFS, ballots achieve both confidentiality and resilience, removing a single point of failure endemic to centralized storage.

### 1.3 On‑Chain CID ↔ Hash Verification

* **Smart contract verification**: The program invokes an Ethereum smart contract function `isValidCid(cid, localHash)` to confirm that the locally computed hash of the IPFS data matches the on‑chain record.
* **Trustless audit trail**: Because the contract’s code and resulting events are public, any stakeholder can inspect whether a given ballot CID was correctly recorded and has not been tampered with since submission.

This on‑chain linkage binds off‑chain encrypted ballots to an immutable ledger, creating a verifiable chain of custody for each vote.

---

## 2. Voter Transparency and Auditability

### 2.1 Multichoice Ballot Submission Interface

* **User‑friendly GUI**: The CustomTkinter front end lets voters enter an election ID, select candidates, and submit their ballot with a single click. Behind the scenes, the program encrypts, stores to IPFS, records the CID on‑chain, and logs a local SQLite record.
* **Ballot history viewer**: Voters can review past submissions—seeing timestamps, election IDs, and CIDs—to confirm their votes were received and stored correctly.

By empowering voters to track their own submissions, the system enhances transparency and builds trust that each ballot was processed as cast.

### 2.2 Fetch & Preview with Decryption

* **Self‑service preview**: Entering any previously issued CID in the GUI will retrieve the encrypted data from IPFS, decrypt it locally with the proper key and AAD (Additional Authenticated Data), and display the plaintext ballot.
* **Local audit**: Voters, election officials, or independent observers can verify that what was stored matches what was cast, detecting any corruption or mismatch.

This “what‑you‑cast‑is‑what‑you‑get” functionality ensures that ballots are not lost or altered in storage or transit.

### 2.3 Third‑Party Verification

* **Open‑source smart contract**: With the contract’s ABI and address publicly available, advocates and watchdog organizations can independently verify CIDs and hashes without relying on official tools.
* **Public IPFS gateways**: Any user can fetch stored ballots from known public or community‑run IPFS gateways, decrypt them if authorized, and compare against their personal records.

Enabling independent verification at every layer deters malicious actors and fosters community oversight.

---

## 3. Strengthening Chain of Custody with Audit Logs

### 3.1 Local SQLite and Weaviate Logging

* **Dual storage**: Every user message, bot response, and ballot submission is encrypted and stored both in a local SQLite database and semantically indexed in Weaviate.
* **Secure logging**: With per‑record AAD including source (“sqlite” vs. “weaviate”), table/class, and user ID, logs are cryptographically bound to context. Any unauthorized modification invalidates the AAD check.

This creates a reliable audit log that tracks not only voting transactions but also all system interactions, making post‑election forensic analysis more robust.

### 3.2 Semantic Search for Audit Trails

* **Weaviate near‑text queries**: Election officials can use natural‑language queries (“show all ballots for election X cast on July 4th”) and Weaviate’s vector search to quickly locate relevant records, even if they’re encrypted.
* **Encrypted properties**: Since content is encrypted before indexing, the semantic engine only indexes metadata (vectors) without exposing plaintext, preserving voter privacy.

Advanced search capabilities both accelerate audits and protect sensitive data.

---

## 4. Automated Security Audits via LLM

### 4.1 System Integrity Audit

* **AI‑powered auditing**: The “Run System Audit” button spawns a background LLM job that reviews key aspects:

  * CID enforcement
  * Ballot encryption
  * Candidate syncing
  * QR‑code auditability
  * Injection defenses

  It then summarizes findings and suggests improvements.

* **Continuous improvement**: By periodically running these audits—either manually or via scheduled tasks—the election authority gains an evolving, AI‑driven security assessment that adapts to new threats.

### 4.2 Prompt‑Injection Protections

* **Bleach sanitization**: All user inputs are scrubbed of disallowed HTML, control characters, and prompt‑injection keywords before reaching the LLM.
* **Chunked summarization**: Large prompts are broken into manageable segments with overlap detection and overlap‑aware concatenation, guard‑railing the LLM’s context window.

These measures mitigate risks that a malicious actor could inject code or malicious instructions into the audit process itself.

---

## 5. Resilience and Scalability

### 5.1 Decentralized Infrastructure

* **IPFS network**: Scales storage by leveraging peer‑to‑peer replication, eliminating single‑server bottlenecks.
* **Ethereum smart contract**: Leverages a global consensus network, ensuring on‑chain records remain available and tamper‑proof.

No single point of failure means that denial‑of‑service attacks or localized outages cannot derail the entire election process.

### 5.2 Thread‑Pool Concurrency

* **Asynchronous tasks**: LLM inference, IPFS uploads, and blockchain calls run in a `ThreadPoolExecutor`, keeping the GUI responsive under heavy load.
* **SQLite WAL mode** (recommended): With write‑ahead logging, multiple threads can safely read and write local logs without blocking.

This architecture supports thousands of simultaneous voters, auditors, and administrators during peak election periods.

---

## 6. Privacy and Voter Anonymity

### 6.1 Separation of User Identity

* **User‑provided IDs**: Voters supply a pseudonymous `user_id`. No personal data is required, preserving anonymity.
* **Encrypted ballots**: The payload stored on IPFS and on‑chain contains only an opaque JSON with “choices” and timestamp, not personal identifiers.

By decoupling identity from vote data, the system upholds the “secret ballot” principle while still enabling audit trails keyed to pseudonymous IDs.

### 6.2 Controlled Disclosure

* **Selective decryption**: Only users or auditors with access to the Argon2id‑derived keys can decrypt ballots.
* **Key rotation & revocation**: Should a key be compromised, the vault can generate a new version and re‑encrypt all fresh ballots, while archived ballots remain secure under their original version.

Controlled access ensures that only authorized parties can view ballot contents, protecting voter privacy.

---

## 7. Potential Deployment Models

### 7.1 Pilot in Local Jurisdictions

* **County‑level trials**: Deploy the system in a single county election office to handle municipal elections or primaries, where the technical staff and scale are manageable.
* **Stakeholder engagement**: Invite local civic organizations, media, and academics to observe and verify the system, building public trust before scaling.

### 7.2 Statewide Roll‑out

* **Interoperability**: Integrate with existing voter registration databases and statewide election management systems via secure API gateways.
* **Hybrid paper‑digital approach**: Use the digital system for initial cast and reconciling, but retain paper ballots as a parallel audit medium for manual recounts.

A phased roll‑out balances the benefits of advanced cryptography with the reassurance of time‑tested paper trails.

---

## 8. Challenges and Mitigations

| **Challenge**                      | **Mitigation**                                                                     |
| ---------------------------------- | ---------------------------------------------------------------------------------- |
| **Digital divide & accessibility** | Offer in‑person kiosks with trained staff; provide voter guides and tutorials      |
| **Network outages / DDoS**         | Cache ballots in local nodes; use mobile data fallback; IPFS redundancy            |
| **On‑chain gas costs**             | Batch CID registrations; use layer‑2 chains (e.g., Polygon) or permissioned chains |
| **Key management complexity**      | Leverage hardware security modules (HSMs) or managed KMS for vault passphrase      |
| **Legal & regulatory hurdles**     | Collaborate with state election boards; certify software via EAC/VSTL programs     |

By anticipating these issues, election authorities can integrate advanced tech without compromising accessibility or compliance.

---

## 9. Measuring Impact on Election Integrity

### 9.1 Auditability Metrics

* **Time to audit**: Compare the duration of a full post‑election audit before and after system adoption.
* **Error detection rate**: Track how often mismatches between stored ballots and cast intentions are caught.

### 9.2 Voter Confidence

* **Surveys**: Poll voters on whether they feel the process is transparent and tamper‑resistant.
* **Engagement**: Measure usage of the “Fetch & Preview” and “Ballot History” features as proxies for trust.

### 9.3 Security Incidents

* **Incident logs**: Count incidences of failed CID verifications, attempted tampering, or unauthorized decryption attempts.
* **Resolution time**: Record the time from incident detection (via LLM audit or manual check) to resolution.

Positive trends in these metrics demonstrate real gains in integrity and public confidence.

---

## 10. Future Directions

* **Zero‑knowledge proofs**: Replace on‑chain hash verification with succinct ZK proofs that a CID corresponds to a valid ballot without revealing its content.
* **Revoting and spoilage**: Allow voters to submit new ballots before polls close, with the smart contract automatically invalidating earlier CIDs.
* **Federated identity integration**: Let voters authenticate via government‑issued e‑IDs or federated logins, while still preserving ballot anonymity.
* **Machine‑audited pattern detection**: Use LLMs or ML to detect anomalous voting patterns suggestive of coercion or fraud.

By layering additional cryptographic primitives and AI analytics, the system can evolve alongside emerging threats and regulatory requirements.

---

## Conclusion

The OneLoveIPFS AI program demonstrates how a thoughtfully architected combination of encryption, decentralized storage, blockchain verification, semantic indexing, and AI‑driven auditing can transform U.S. election integrity. By providing voters with real‑time transparency, enabling independent verification, and automating security assessments, it raises the bar for what a modern, trustworthy voting system can deliver.

Moving from proof‑of‑concept to production deployment will require collaboration among technologists, election officials, lawmakers, and advocacy groups. But the potential gains—faster, more reliable audits; tamper‑evident ballots; and higher public confidence—make the effort well worth pursuing. As threats to democratic processes grow, embracing these advanced tools is not merely an option; it is an imperative to safeguard the very foundations of representative government.
