
Mini project for **Distributed Computing**. A discrete-event simulation of Maekawa's quorum-based
mutual exclusion algorithm, with deadlock handling, safety checking and a comparison against Ricart-Agrawala.

## Idea
Instead of asking *all* N-1 nodes for permission, a node asks only the members of its **request set (quorum)**.
Quorums are built on a √N × √N grid (row + column of the node, size ≈ 2√N − 1) so that **any two quorums intersect**.
A node enters the critical section (CS) only after collecting a REPLY (vote) from every quorum member.
Each node gives out one vote at a time, so two nodes can never both hold all their votes ⇒ **mutual exclusion**.

## Messages
| Message | Meaning |
|---|---|
| REQUEST(ts,id) | ask a quorum member for its vote (Lamport timestamp gives priority) |
| REPLY | vote granted |
| RELEASE | CS finished, vote free again |
| FAILED | "your request has lower priority than someone I voted for / queued" |
| INQUIRE | voter asks the current holder "can you give my vote back?" |
| YIELD | requester returns the vote (it already knows it has failed) – breaks deadlock |

## Run
```bash
pip install matplotlib   # only needed for the charts
python maekawa.py --n 9 --rounds 3 --seed 1 --scaling   # N must be a perfect square: 4, 9, 16, 25...
python test_maekawa.py                                  # stress test
```
Outputs: console log of every request/entry/exit, `outputs/cs_timeline.png`, `outputs/message_complexity.png`.

## Results
* Mutual exclusion asserted on every CS entry – never violated.
* 360 random schedules (N = 4…49): no deadlock, every node completes all its CS entries.
* Messages per CS entry between 3(K−1) (no contention) and 5(K−1) (heavy contention), K = quorum size ≈ 2√N − 1, i.e. **O(√N)**, vs. 2(N−1) = O(N) for Ricart-Agrawala. Measured: N=36 → 40 msgs/CS vs 70.

## Bug found & fixed during development
Without sending FAILED to lower-priority queued requests after a vote is re-granted, a requester never learns it is
losing, keeps its other votes and the system **deadlocks** (found with N=25, seed=5). See `grant_next()`.

## Files
`maekawa.py` simulator · `test_maekawa.py` tests · `outputs/` charts
