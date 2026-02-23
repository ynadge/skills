---
name: sapiom-verify
description: >
  Send and verify phone number or email address verification codes using
  Sapiom's API — no Twilio account, no SendGrid account, no vendor onboarding.
  Use when an agent or app needs to verify that a user owns a phone number or
  email address via a 6-digit OTP code flow. Do NOT use for non-verification
  messaging (e.g. notifications or marketing SMS — this is OTP only).
license: MIT
metadata:
  version: "1.0.0"
---

# User Verification via Sapiom

Sapiom routes phone and email verification through **Prelude** — a verification
service with global SMS delivery and built-in fraud prevention. No separate
Prelude account required.

## When to Apply This Skill

Use this skill when:
- An agent needs to verify a user owns a phone number or email address
- Building sign-up or login flows that require OTP verification
- Adding 2FA to an existing application
- Confirming contact details before sending communications

## How the Flow Works

Verification is always two steps:

1. **Send** — POST to `/verifications` with the target phone or email. The user
   receives a 6-digit code. You get back a `verificationRequestId`.
2. **Check** — POST to `/verifications/check` with the `verificationRequestId`
   and the code the user entered. The check endpoint is **free**.

Codes expire after **10 minutes**. If the user misses it, send a new request.

## Setup

```bash
npm install @sapiom/fetch
```

```bash
export SAPIOM_API_KEY="your_api_key_here"
```

```typescript
import { createFetch } from "@sapiom/fetch";

const fetch = createFetch({
  apiKey: process.env.SAPIOM_API_KEY!,
  agentName: "my-agent",
  serviceName: "SMS Verification",
});

const BASE_URL = "https://prelude.services.sapiom.ai";
```

## Phone Number Verification

Phone numbers must be in **E.164 format** — international format with country
code and no spaces (e.g. `+15551234567` for US, `+442071234567` for UK).

```typescript
// Step 1: Send the code
const sendResponse = await fetch(`${BASE_URL}/verifications`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    target: {
      type: "phone_number",
      value: "+15551234567",
    },
  }),
});

const { id: verificationRequestId } = await sendResponse.json();
// Store verificationRequestId — you need it for the check step
```

```typescript
// Step 2: Check the code (after the user submits it)
const checkResponse = await fetch(`${BASE_URL}/verifications/check`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    verificationRequestId,
    code: userEnteredCode, // 4–8 digit string
  }),
});

const { status } = await checkResponse.json();
const isVerified = status === "success";
```

## Email Verification

Same flow, different `target.type`:

```typescript
const sendResponse = await fetch(`${BASE_URL}/verifications`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    target: {
      type: "email_address",
      value: "user@example.com",
    },
  }),
});

const { id: verificationRequestId } = await sendResponse.json();
```

The check step is identical to the phone flow.

## Complete Reusable Function

```typescript
import { createFetch } from "@sapiom/fetch";

const fetch = createFetch({ apiKey: process.env.SAPIOM_API_KEY! });
const BASE_URL = "https://prelude.services.sapiom.ai";

type TargetType = "phone_number" | "email_address";

// Call this when the user submits their phone/email
async function sendVerificationCode(
  type: TargetType,
  value: string
): Promise<string> {
  const response = await fetch(`${BASE_URL}/verifications`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ target: { type, value } }),
  });

  if (!response.ok) {
    const err = await response.json();
    throw new Error(`Failed to send code: ${response.status} ${JSON.stringify(err)}`);
  }

  const data = await response.json();
  return data.id; // verificationRequestId — store this in session/state
}

// Call this when the user submits their code
async function checkVerificationCode(
  verificationRequestId: string,
  code: string
): Promise<boolean> {
  const response = await fetch(`${BASE_URL}/verifications/check`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ verificationRequestId, code }),
  });

  if (!response.ok) {
    const status = response.status;
    if (status === 410) throw new Error("Verification code expired — send a new one");
    if (status === 429) throw new Error("Too many attempts — ask user to wait");
    throw new Error(`Check failed: ${status}`);
  }

  const data = await response.json();
  return data.status === "success";
}

// Usage
const requestId = await sendVerificationCode("phone_number", "+15551234567");
// ... user receives SMS and enters code in your UI ...
const verified = await checkVerificationCode(requestId, userEnteredCode);
```

## Check Response Statuses

| Status | Meaning |
|--------|---------|
| `success` | Code is correct — user is verified |
| `failure` | Code is incorrect — let user retry |
| `pending` | Verification still in progress |

## Error Handling

| Code | Meaning | What to do |
|------|---------|------------|
| 400 | Invalid phone/email format or bad code format | Check E.164 format for phone; validate input |
| 402 | Sapiom payment required | Ensure SDK is wrapping your fetch — never call Prelude directly |
| 404 | Verification request not found | `verificationRequestId` is wrong or already used |
| 410 | Code expired | Send a fresh verification request |
| 422 | Wrong code entered | Prompt user to try again |
| 429 | Too many check attempts | Rate limited — show a cooldown message |

**Important:** The 402 error means you called the Prelude endpoint directly
without the Sapiom SDK wrapping it. Always use `createFetch` from `@sapiom/fetch`
(or the axios/node-http equivalents) — never call `https://prelude.services.sapiom.ai`
with plain fetch or axios.

## Pricing

| Operation | Cost |
|-----------|------|
| Send verification code (SMS or email) | $0.015 |
| Check verification code | Free |

The check step is free, so failed attempts don't cost anything. Only successful
sends are billed.