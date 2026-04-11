# Token Generation Guide

Server-side embed token generation for the Affonso embedded dashboard. Tokens authenticate partners and are valid for 30 minutes.

## Security Rules

- **Always generate tokens server-side** — never expose your API key to the client
- **Generate a fresh token on every page load** — do not cache or reuse tokens
- **Store your API key securely** — use environment variables, not source code

## API Endpoint

```
POST https://api.affonso.io/v1/embed/token

Headers:
  Authorization: Bearer sk_live_YOUR_API_KEY
  Content-Type: application/json
```

## Request Body

```json
{
  "programId": "YOUR_PROGRAM_ID",
  "partner": {
    "email": "partner@example.com",
    "name": "Partner Name",
    "image": "https://example.com/avatar.jpg"
  },
  "groupId": "GROUP_ID"
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `programId` | Yes | Your Affonso program ID |
| `partner.email` | Yes | Partner's email address — used to identify or auto-create the partner |
| `partner.name` | No | Display name shown in the dashboard |
| `partner.image` | No | Avatar URL shown in the dashboard |
| `groupId` | No | Assign partner to a specific commission tier/group |

## Response

```json
{
  "success": true,
  "data": {
    "publicToken": "eyJhbGci...",
    "expiresAt": "2025-01-01T01:00:00.000Z",
    "link": "https://yoursite.com?atp=TRACKING_ID",
    "portalUrl": "https://affiliate.yoursite.com",
    "partnershipStatus": "APPROVED"
  }
}
```

| Field | Description |
|-------|-------------|
| `publicToken` | Token to use in the iframe URL |
| `expiresAt` | Token expiration timestamp (30 minutes from creation) |
| `link` | The partner's unique referral link |
| `portalUrl` | Full portal URL (if custom domain configured) |
| `partnershipStatus` | `APPROVED` — partner is active, `PENDING` — partner awaits approval |

## Auto-Creation Behavior

If the partner email does not exist in your program, the API will **automatically create a new partner**. The initial `partnershipStatus` depends on your program settings:

- **Auto-approve enabled**: Status will be `APPROVED`
- **Auto-approve disabled**: Status will be `PENDING` — the partner must be approved manually before they can access the full dashboard

## Next.js (App Router)

```typescript
// app/api/embed-token/route.ts
import { NextResponse } from 'next/server';
import { getServerSession } from 'next-auth';

export async function POST() {
  const session = await getServerSession();
  if (!session?.user?.email) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const response = await fetch('https://api.affonso.io/v1/embed/token', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${process.env.AFFONSO_API_KEY}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      programId: process.env.AFFONSO_PROGRAM_ID,
      partner: {
        email: session.user.email,
        name: session.user.name,
        image: session.user.image,
      },
    }),
  });

  const data = await response.json();
  return NextResponse.json(data);
}
```

## Next.js (Pages Router)

```typescript
// pages/api/embed-token.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { getSession } from 'next-auth/react';

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const session = await getSession({ req });
  if (!session?.user?.email) {
    return res.status(401).json({ error: 'Unauthorized' });
  }

  const response = await fetch('https://api.affonso.io/v1/embed/token', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${process.env.AFFONSO_API_KEY}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      programId: process.env.AFFONSO_PROGRAM_ID,
      partner: {
        email: session.user.email,
        name: session.user.name,
        image: session.user.image,
      },
    }),
  });

  const data = await response.json();
  res.status(response.status).json(data);
}
```

## Express / Node.js

```javascript
// routes/embed-token.js
const express = require('express');
const router = express.Router();

router.post('/embed-token', async (req, res) => {
  // Assumes auth middleware sets req.user
  if (!req.user?.email) {
    return res.status(401).json({ error: 'Unauthorized' });
  }

  const response = await fetch('https://api.affonso.io/v1/embed/token', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${process.env.AFFONSO_API_KEY}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      programId: process.env.AFFONSO_PROGRAM_ID,
      partner: {
        email: req.user.email,
        name: req.user.name,
      },
    }),
  });

  const data = await response.json();
  res.status(response.status).json(data);
});

module.exports = router;
```

## Python (FastAPI)

```python
# routes/embed_token.py
import os
import httpx
from fastapi import APIRouter, Depends, HTTPException
from app.auth import get_current_user

router = APIRouter()

@router.post("/embed-token")
async def create_embed_token(user = Depends(get_current_user)):
    if not user.email:
        raise HTTPException(status_code=401, detail="Unauthorized")

    async with httpx.AsyncClient() as client:
        response = await client.post(
            "https://api.affonso.io/v1/embed/token",
            headers={
                "Authorization": f"Bearer {os.environ['AFFONSO_API_KEY']}",
                "Content-Type": "application/json",
            },
            json={
                "programId": os.environ["AFFONSO_PROGRAM_ID"],
                "partner": {
                    "email": user.email,
                    "name": user.name,
                },
            },
        )

    return response.json()
```

## Ruby on Rails

```ruby
# app/controllers/api/embed_tokens_controller.rb
class Api::EmbedTokensController < ApplicationController
  before_action :authenticate_user!

  def create
    response = Net::HTTP.post(
      URI("https://api.affonso.io/v1/embed/token"),
      {
        programId: ENV["AFFONSO_PROGRAM_ID"],
        partner: {
          email: current_user.email,
          name: current_user.name
        }
      }.to_json,
      {
        "Authorization" => "Bearer #{ENV['AFFONSO_API_KEY']}",
        "Content-Type" => "application/json"
      }
    )

    render json: JSON.parse(response.body), status: response.code.to_i
  end
end
```

## PHP (Laravel)

```php
// app/Http/Controllers/EmbedTokenController.php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Http;

class EmbedTokenController extends Controller
{
    public function __invoke(Request $request)
    {
        $user = $request->user();

        $response = Http::withHeaders([
            'Authorization' => 'Bearer ' . config('services.affonso.api_key'),
        ])->post('https://api.affonso.io/v1/embed/token', [
            'programId' => config('services.affonso.program_id'),
            'partner' => [
                'email' => $user->email,
                'name' => $user->name,
            ],
        ]);

        return response()->json($response->json(), $response->status());
    }
}
```

## Environment Variables

Add these to your `.env` file:

```
AFFONSO_API_KEY=sk_live_your_api_key_here
AFFONSO_PROGRAM_ID=your_program_id_here
```

## Error Handling

| Status | Code | Cause |
|--------|------|-------|
| 401 | `UNAUTHORIZED` | Invalid or missing API key |
| 400 | `VALIDATION_ERROR` | Missing required fields (`programId`, `partner.email`) |
| 404 | `NOT_FOUND` | Invalid program ID |
| 429 | `RATE_LIMITED` | Too many requests — implement exponential backoff |
