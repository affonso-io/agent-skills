# Framework Examples

Complete end-to-end examples for embedding the Affonso affiliate dashboard in your application. Each example includes the server-side token endpoint and the client-side iframe component.

## Next.js (App Router)

### Server: API Route

```typescript
// app/api/embed-token/route.ts
import { NextResponse } from 'next/server';
import { getServerSession } from 'next-auth';
import { authOptions } from '@/lib/auth';

export async function POST() {
  const session = await getServerSession(authOptions);
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

### Client: Dashboard Component

```tsx
// components/AffiliateDashboard.tsx
'use client';

import { useEffect, useState } from 'react';

interface EmbedConfig {
  theme?: 'light' | 'dark' | 'system';
  bg?: string;
  lang?: string;
  showHeader?: boolean;
  showRewards?: boolean;
  showReports?: boolean;
  showResources?: boolean;
  enableQRCode?: boolean;
  padding?: boolean;
}

export function AffiliateDashboard({ config = {} }: { config?: EmbedConfig }) {
  const [token, setToken] = useState<string | null>(null);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    async function fetchToken() {
      const res = await fetch('/api/embed-token', { method: 'POST' });
      const data = await res.json();
      if (data.success) {
        setToken(data.data.publicToken);
      } else {
        setError(data.error?.message || 'Failed to load dashboard');
      }
    }
    fetchToken();
  }, []);

  if (error) return <div>Error: {error}</div>;
  if (!token) return <div>Loading dashboard...</div>;

  const params = new URLSearchParams({ token });
  Object.entries(config).forEach(([key, value]) => {
    if (value !== undefined) params.set(key, String(value));
  });

  return (
    <iframe
      src={`https://affonso.io/embed/referrals?${params.toString()}`}
      style={{ width: '100%', height: '100%', minHeight: '600px', border: 'none' }}
      allow="clipboard-write"
      title="Affiliate Dashboard"
    />
  );
}
```

### Usage in a Page

```tsx
// app/dashboard/referrals/page.tsx
import { AffiliateDashboard } from '@/components/AffiliateDashboard';

export default function ReferralsPage() {
  return (
    <div style={{ height: '100vh' }}>
      <AffiliateDashboard
        config={{
          theme: 'dark',
          bg: '0f172a',
          showResources: false,
          padding: false,
        }}
      />
    </div>
  );
}
```

## Next.js (Pages Router)

### Server: API Route

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

### Client: Dashboard Component

```tsx
// components/AffiliateDashboard.tsx
import { useEffect, useState } from 'react';

export function AffiliateDashboard({ theme = 'light', lang = 'en' }) {
  const [token, setToken] = useState<string | null>(null);

  useEffect(() => {
    fetch('/api/embed-token', { method: 'POST' })
      .then(res => res.json())
      .then(data => {
        if (data.success) setToken(data.data.publicToken);
      });
  }, []);

  if (!token) return <div>Loading...</div>;

  const src = `https://affonso.io/embed/referrals?token=${token}&theme=${theme}&lang=${lang}`;

  return (
    <iframe
      src={src}
      style={{ width: '100%', height: '100%', minHeight: '600px', border: 'none' }}
      allow="clipboard-write"
      title="Affiliate Dashboard"
    />
  );
}
```

## Express / Node.js

### Server

```javascript
// server.js
const express = require('express');
const app = express();

app.post('/api/embed-token', async (req, res) => {
  // Replace with your auth middleware
  const user = req.user;
  if (!user?.email) {
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
      partner: { email: user.email, name: user.name },
    }),
  });

  const data = await response.json();
  res.json(data);
});
```

### Client (Vanilla HTML)

```html
<div id="affiliate-dashboard" style="width: 100%; height: 600px;"></div>

<script>
  async function loadDashboard() {
    const res = await fetch('/api/embed-token', { method: 'POST' });
    const data = await res.json();

    if (data.success) {
      const iframe = document.createElement('iframe');
      iframe.src = `https://affonso.io/embed/referrals?token=${data.data.publicToken}&theme=light`;
      iframe.style.cssText = 'width:100%;height:100%;border:none;';
      iframe.allow = 'clipboard-write';
      iframe.title = 'Affiliate Dashboard';
      document.getElementById('affiliate-dashboard').appendChild(iframe);
    }
  }

  loadDashboard();
</script>
```

## Python (FastAPI)

### Server

```python
# main.py
import os
import httpx
from fastapi import FastAPI, Depends, HTTPException
from fastapi.responses import HTMLResponse
from app.auth import get_current_user

app = FastAPI()

@app.post("/api/embed-token")
async def create_embed_token(user=Depends(get_current_user)):
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

### Server

```ruby
# config/routes.rb
post '/api/embed_token', to: 'embed_tokens#create'

# app/controllers/embed_tokens_controller.rb
class EmbedTokensController < ApplicationController
  before_action :authenticate_user!

  def create
    uri = URI("https://api.affonso.io/v1/embed/token")
    http = Net::HTTP.new(uri.host, uri.port)
    http.use_ssl = true

    request = Net::HTTP::Post.new(uri)
    request["Authorization"] = "Bearer #{ENV['AFFONSO_API_KEY']}"
    request["Content-Type"] = "application/json"
    request.body = {
      programId: ENV["AFFONSO_PROGRAM_ID"],
      partner: {
        email: current_user.email,
        name: current_user.name
      }
    }.to_json

    response = http.request(request)
    render json: JSON.parse(response.body), status: response.code.to_i
  end
end
```

### Client (ERB View)

```erb
<!-- app/views/dashboard/referrals.html.erb -->
<div id="affiliate-dashboard" style="width: 100%; height: 600px;"></div>

<script>
  fetch('/api/embed_token', { method: 'POST', headers: { 'X-CSRF-Token': document.querySelector('meta[name="csrf-token"]').content } })
    .then(res => res.json())
    .then(data => {
      if (data.success) {
        const iframe = document.createElement('iframe');
        iframe.src = `https://affonso.io/embed/referrals?token=${data.data.publicToken}`;
        iframe.style.cssText = 'width:100%;height:100%;border:none;';
        iframe.allow = 'clipboard-write';
        document.getElementById('affiliate-dashboard').appendChild(iframe);
      }
    });
</script>
```

## PHP (Laravel)

### Server

```php
// routes/api.php
Route::post('/embed-token', EmbedTokenController::class)->middleware('auth');

// app/Http/Controllers/EmbedTokenController.php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Http;

class EmbedTokenController extends Controller
{
    public function __invoke(Request $request)
    {
        $response = Http::withHeaders([
            'Authorization' => 'Bearer ' . config('services.affonso.api_key'),
        ])->post('https://api.affonso.io/v1/embed/token', [
            'programId' => config('services.affonso.program_id'),
            'partner' => [
                'email' => $request->user()->email,
                'name' => $request->user()->name,
            ],
        ]);

        return response()->json($response->json(), $response->status());
    }
}
```

### Config

```php
// config/services.php
'affonso' => [
    'api_key' => env('AFFONSO_API_KEY'),
    'program_id' => env('AFFONSO_PROGRAM_ID'),
],
```

### Client (Blade View)

```blade
<!-- resources/views/dashboard/referrals.blade.php -->
<div id="affiliate-dashboard" style="width: 100%; height: 600px;"></div>

<script>
  fetch('/api/embed-token', {
    method: 'POST',
    headers: { 'X-CSRF-TOKEN': '{{ csrf_token() }}' }
  })
    .then(res => res.json())
    .then(data => {
      if (data.success) {
        const iframe = document.createElement('iframe');
        iframe.src = `https://affonso.io/embed/referrals?token=${data.data.publicToken}`;
        iframe.style.cssText = 'width:100%;height:100%;border:none;';
        iframe.allow = 'clipboard-write';
        document.getElementById('affiliate-dashboard').appendChild(iframe);
      }
    });
</script>
```

## Environment Variables

All frameworks need these in their `.env`:

```
AFFONSO_API_KEY=sk_live_your_api_key_here
AFFONSO_PROGRAM_ID=your_program_id_here
```
