# PlayTesterHub MCP

Google Play closed testing: order 12 real Android testers for 14 days and follow the test.

- **Address:** `https://playtesterhub.com/api/mcp` (Streamable HTTP)
- **Auth:** OAuth (sign in with Google on PlayTesterHub, then Allow), or a personal API key created in your dashboard › "Your AI agents" (header `Authorization: Bearer pth_...`).
- **Docs:** https://playtesterhub.com/en/api-mcp

## What your agent can do

- Create an order and get a Stripe payment link for a human to pay (nothing is charged before that).
- Follow campaigns: status, test day out of 14, the team's message.
- Read reports (recaps and final report on Premium plans, daily per-device reports on Premium+) as text and JSON.
- Read tips to improve the app (all plans).
- Confirm that the tester group was added in Play Console.
- Ask the team to install a new version on the 12 test phones.
- Read and send messages to the team, read notifications.

## Claude Code with an API key

```bash
claude mcp add --transport http playtesterhub https://playtesterhub.com/api/mcp \
  --header "Authorization: Bearer pth_YOUR_KEY"
```

Revoke access at any time in your dashboard › "Your AI agents".
