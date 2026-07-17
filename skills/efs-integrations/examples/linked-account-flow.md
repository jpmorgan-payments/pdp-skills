# Example: linked account (recipient) flow

Reference walkthrough distilled from the OSS [Linked Accounts recipe](https://github.com/jpmorgan-payments/embedded-finance/blob/main/embedded-components/docs/LINKED_ACCOUNTS_RECIPE.md). Adapt field names to your OAS version.

## 1. Add linked account

`POST /recipients` with a body shaped like:

```json
{
  "type": "LINKED_ACCOUNT",
  "partyDetails": {
    "clientId": "<clientId>"
  },
  "accountDetails": {
    "accountNumber": "********",
    "routingNumber": "021000021",
    "accountType": "CHECKING"
  }
}
```

Persist `recipientId` and initial `status` from the `201` response.

## 2. Branch on status

| Status | Platform action |
| --- | --- |
| `ACTIVE` (or equivalent verified) | Enable payouts to this account |
| `MICRODEPOSITS_INITIATED` | Tell the user to watch their bank for two small deposits; collect amounts when ready |
| Failed / rejected | Show reason; allow retry per API rules |

## 3. Verify microdeposits

`POST /recipients/{recipientId}/verify-microdeposit` with the two amounts (currency/units per OAS).

On success, refresh with `GET /recipients/{recipientId}` before enabling payouts.

## 4. Optional notifications

Subscribe to `RECIPIENT_ACCOUNT_VALIDATION` so reminders/expiry don't depend only on the user returning to the app — see `references/notifications.md`.

## 5. Minimal service sketch

```ts
// Pseudocode — use your project's EF&S HTTP client
export async function linkExternalAccount(input: {
  clientId: string;
  routingNumber: string;
  accountNumber: string;
  accountType: "CHECKING" | "SAVINGS";
}) {
  return efsClient.request("POST", "/recipients", {
    type: "LINKED_ACCOUNT",
    partyDetails: { clientId: input.clientId },
    accountDetails: {
      routingNumber: input.routingNumber,
      accountNumber: input.accountNumber,
      accountType: input.accountType,
    },
  });
}

export async function verifyMicrodeposits(
  recipientId: string,
  amounts: [number, number],
) {
  return efsClient.request(
    "POST",
    `/recipients/${recipientId}/verify-microdeposit`,
    { amounts /* shape per OAS */ },
  );
}
```

Confirm exact paths and payload property names against the OpenAPI spec before shipping.
