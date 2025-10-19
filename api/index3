export default async function handler(req, res) {
  // Enable CORS
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'POST, OPTIONS');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');

  // Handle OPTIONS (preflight) requests
  if (req.method === 'OPTIONS') {
    return res.status(200).end();
  }

  // Only accept POST requests
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    // Get API key from environment variable
    const apiKey = process.env.GEMINI_API_KEY;
    if (!apiKey) {
      return res.status(500).json({ error: 'API key not configured' });
    }

    const { messages, model = 'gemini-2.5-pro' } = req.body;

    if (!messages || !Array.isArray(messages)) {
      return res.status(400).json({ error: 'Messages array required' });
    }

    // Format messages for Gemini API
    const contents = messages
      .filter(msg => msg.role !== 'system')
      .map(msg => ({
        role: msg.role === 'user' ? 'user' : 'model',
        parts: [{ text: msg.content }]
      }));

    // Extract system prompt if present
    const systemPrompt = messages.find(msg => msg.role === 'system')?.content || '';

    // Call Gemini API
    const geminiUrl = `https://generativelanguage.googleapis.com/v1beta/models/${model}:generateContent?key=${apiKey}`;

    const geminiResponse = await fetch(geminiUrl, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        system_instruction: systemPrompt,
        contents: contents,
        generationConfig: {
          temperature: 0.7,
          maxOutputTokens: 1024,
        },
      }),
    });

    const geminiData = await geminiResponse.json();

    // Handle API errors
    if (!geminiResponse.ok) {
      console.error('Gemini API error:', geminiData);
      return res.status(geminiResponse.status).json({
        error: geminiData.error?.message || 'Gemini API error',
      });
    }

    // Extract response text
    const responseText = geminiData.candidates?.[0]?.content?.parts?.[0]?.text || 'No response';

    // Format response in OpenAI-compatible format for JanitorAI
    return res.status(200).json({
      choices: [
        {
          message: {
            role: 'assistant',
            content: responseText,
          },
        },
      ],
    });
  } catch (error) {
    console.error('Proxy error:', error);
    return res.status(500).json({
      error: 'Internal server error',
      message: error.message,
    });
  }
}
