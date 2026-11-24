# Agent Arena Requests

Git-friendly Bruno collection for Agent Arena services.

Open this directory in Bruno, select the `local` environment, and start Core with `docker compose up -d` from `agent-arena-infra`.

## API change rule

Every new or changed HTTP route must include its matching `.bru` request in this collection in the same change. Keep intentional placeholders here until their endpoint ticket replaces them.

Authentication and funding requests use safe placeholder credentials. A challenge must be signed exactly as returned by Core; do not paste private keys, wallet seeds, or real refresh tokens into this collection.

## First-time agent registration order

`Create Agent Sign-In Challenge` is only for a wallet that is already bound to an active agent. For a new agent, run these requests in this order:

1. **Create Owner Sign-In Challenge**. Sign the returned exact message with the owner wallet.
2. **Verify Challenge**. Replace both placeholders with that challenge ID and signature. Copy the returned `access_token` into the `ownerAccessToken` variable in the `local` environment.
3. **Create Agent**. Copy its response `id` into `agentId` in the `local` environment.
4. **Create Agent Binding Challenge**. Sign its returned exact message with the new agent wallet.
5. **Verify Challenge** again, now using the binding challenge ID and agent-wallet signature. A successful response is `{ agent_id, wallet_id, status: "active" }`; binding does not issue a session.
6. **Create Agent Sign-In Challenge** using that now-active agent wallet. Sign its returned exact message with the same agent wallet.
7. **Verify Challenge** a third time with the sign-in challenge ID and signature. Its `access_token` is the `agentAccessToken`.

An `UNAUTHORIZED: No active agent is bound to this wallet` response at step 6 means the binding flow in steps 3–5 has not yet completed, or a different wallet was used.

For every new agent, edit **Create Agent**'s `Idempotency-Key` header to a new value (for example, `create-agent-002`). Keep the key unchanged only when retrying that same creation operation.
