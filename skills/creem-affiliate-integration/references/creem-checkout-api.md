# Creem Checkout API Integration

For users implementing affiliate tracking with the Creem Checkout API.

## Implementation

Pass the `affonso_referral` cookie as metadata to your Creem checkout request:

```javascript
import { cookies } from 'next/headers';
import axios from 'axios';

// Get Affonso referral cookie (Next.js 15+)
// For Next.js 13-14: const cookieStore = cookies(); (without await)
const cookieStore = await cookies();
const affonsoReferral = cookieStore.get('affonso_referral')?.value || '';

// Create Creem checkout with referral metadata
const response = await axios.post(
  'https://api.creem.io/v1/checkouts',
  {
    request_id: 'your-request-id',
    product_id: 'prod_your-product-id',
    metadata: {
      affonso_referral: affonsoReferral, // Add this field
    },
  },
  {
    headers: {
      'x-api-key': process.env.CREEM_API_KEY,
    },
  }
);
```

## Testing

1. Visit your site with `?atp=test` (test affiliate parameter)
2. Check browser DevTools → Application → Cookies → look for `affonso_referral`
3. Trigger your checkout flow locally
4. Check your server logs/terminal - the Creem request should show:
   ```
   metadata: { affonso_referral: "cmdhq6ayf..." }
   ```
5. If the metadata appears in your logs, you're all set!

**Note:** Use 100% discount codes for live testing without payments, or test locally with server logs - Creem test mode won't appear in Affonso.

## Framework-Specific Examples

### Node.js/Express
```javascript
const axios = require('axios');

const affonsoReferral = req.cookies.affonso_referral || '';

const response = await axios.post(
  'https://api.creem.io/v1/checkouts',
  {
    request_id: 'your-request-id',
    product_id: 'prod_your-product-id',
    metadata: {
      affonso_referral: affonsoReferral,
    },
  },
  {
    headers: {
      'x-api-key': process.env.CREEM_API_KEY,
    },
  }
);
```

### Python/Flask
```python
import requests

affonso_referral = request.cookies.get('affonso_referral', '')

response = requests.post(
    'https://api.creem.io/v1/checkouts',
    json={
        'request_id': 'your-request-id',
        'product_id': 'prod_your-product-id',
        'metadata': {
            'affonso_referral': affonso_referral,
        },
    },
    headers={
        'x-api-key': os.environ['CREEM_API_KEY'],
    },
)
```

### Python/Django
```python
import requests

affonso_referral = request.COOKIES.get('affonso_referral', '')

response = requests.post(
    'https://api.creem.io/v1/checkouts',
    json={
        'request_id': 'your-request-id',
        'product_id': 'prod_your-product-id',
        'metadata': {
            'affonso_referral': affonso_referral,
        },
    },
    headers={
        'x-api-key': os.environ['CREEM_API_KEY'],
    },
)
```

### Ruby/Rails
```ruby
require 'net/http'
require 'json'

affonso_referral = cookies[:affonso_referral] || ''

uri = URI('https://api.creem.io/v1/checkouts')
http = Net::HTTP.new(uri.host, uri.port)
http.use_ssl = true

request = Net::HTTP::Post.new(uri)
request['x-api-key'] = ENV['CREEM_API_KEY']
request['Content-Type'] = 'application/json'
request.body = {
  request_id: 'your-request-id',
  product_id: 'prod_your-product-id',
  metadata: {
    affonso_referral: affonso_referral,
  },
}.to_json

response = http.request(request)
```

### PHP
```php
$affonsoReferral = $_COOKIE['affonso_referral'] ?? '';

$response = file_get_contents('https://api.creem.io/v1/checkouts', false, stream_context_create([
    'http' => [
        'method' => 'POST',
        'header' => implode("\r\n", [
            'Content-Type: application/json',
            'x-api-key: ' . getenv('CREEM_API_KEY'),
        ]),
        'content' => json_encode([
            'request_id' => 'your-request-id',
            'product_id' => 'prod_your-product-id',
            'metadata' => [
                'affonso_referral' => $affonsoReferral,
            ],
        ]),
    ],
]));
```
