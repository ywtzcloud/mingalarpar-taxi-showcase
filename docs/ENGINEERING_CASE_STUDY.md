# Engineering Case Study — Mingalarpar Taxi

## Problem

Dispatch systems are exposed to timing races: multiple drivers can see related events, clients may be stale, network responses can arrive out of order and background workers may change order state while users are interacting with it.

## Engineering response

1. **Server-side eligibility.** Backend logic validates whether a driver is actually eligible for a protected dispatch action.
2. **Explicit lifecycle state.** Order states are treated as business transitions with controlled conditions.
3. **Controlled recovery.** Freeze and recovery paths repair inconsistent state without silently bypassing ownership rules.
4. **Authentication generation fencing.** Late background results from an earlier login cannot mutate the active session.
5. **Repeat-safe financial behavior.** Finance and settlement use stable operation identity.
6. **Separate business truth from notification transport.** FCM and realtime are delivery mechanisms, not authoritative state.

## Engineering value demonstrated

- native Android architecture
- distributed-state reasoning
- dispatch and concurrency control
- backend authorization
- recovery-state design
- realtime synchronization
- finance integrity
- secure release engineering
