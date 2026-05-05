module.exports = (req, res) => {
  // CORS - must be set before anything else
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'POST, OPTIONS, GET');
  res.setHeader('Access-Control-Allow-Headers', '*');
  res.setHeader('Access-Control-Max-Age', '86400');

  // Handle OPTIONS preflight immediately
  if (req.method === 'OPTIONS') {
    res.status(204).end();
    return;
  }

  // Only allow POST
  if (req.method !== 'POST') {
    res.status(405).json({ error: 'Method not allowed' });
    return;
  }

  // Parse body
  let body = '';
  req.on('data', chunk => { body += chunk.toString(); });
  req.on('end', () => {
    let prompt, max_tokens;
    try {
      const parsed = JSON.parse(body);
      prompt = parsed.prompt;
      max_tokens = parsed.max_tokens || 1200;
    } catch (e) {
      res.status(400).json({ error: 'Invalid JSON body' });
      return;
    }

    if (!prompt) {
      res.status(400).json({ error: 'Missing prompt' });
      return;
    }

    const apiKey = process.env.ANTHROPIC_API_KEY;
    if (!apiKey) {
      res.status(500).json({ error: 'Server configuration error' });
      return;
    }

    const https = require('https');
    const payload = JSON.stringify({
      model: 'claude-sonnet-4-20250514',
      max_tokens: max_tokens,
      messages: [{ role: 'user', content: prompt }]
    });

    const options = {
      hostname: 'api.anthropic.com',
      path: '/v1/messages',
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-api-key': apiKey,
        'anthropic-version': '2023-06-01',
        'Content-Length': Buffer.byteLength(payload)
      }
    };

    const apiReq = require('https').request(options, apiRes => {
      let data = '';
      apiRes.on('data', chunk => { data += chunk; });
      apiRes.on('end', () => {
        try {
          const parsed = JSON.parse(data);
          if (!parsed.content) {
            res.status(500).json({ error: 'Bad response from AI', raw: data.substring(0, 200) });
            return;
          }
          const text = parsed.content.map(c => c.text || '').join('');
          res.status(200).json({ text: text });
        } catch (e) {
          res.status(500).json({ error: 'Parse failed: ' + e.message });
        }
      });
    });

    apiReq.on('error', err => {
      res.status(500).json({ error: 'Request failed: ' + err.message });
    });

    apiReq.write(payload);
    apiReq.end();
  });
};
