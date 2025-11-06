import OpenAI from "openai";

const client = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

function sanitizeInput(text){
  if(!text || typeof text !== 'string') return '';
  return text.trim().slice(0, 3000);
}

function buildPrompt(text){
  return `You are a gentle, therapeutic assistant who reflects on dreams in a kind, non-clinical tone.
Read the dream text below and return a JSON object EXACTLY in this format:

{
  "keywords": ["kw1","kw2","kw3"],
  "mood": "简短情绪标签，例如：焦虑 / 温柔 / 迷惘 / 平静",
  "summary": "一句温柔的 1-3 句解读与一个温和的自我反思问题"
}

Dream text:
"""${text}"""
`;
}

export default async function handler(req, res){
  try{
    if(req.method !== 'POST') return res.status(405).json({ error: 'Only POST' });
    const { text } = req.body || {};
    const plain = sanitizeInput(text);
    if(!plain) return res.status(400).json({ error: 'No dream text provided' });

    const prompt = buildPrompt(plain);

    const response = await client.responses.create({
      model: 'gpt-4o-mini',
      input: prompt,
      temperature: 0.6,
      max_output_tokens: 400
    });

    // extract text
    let raw = '';
    if(response.output && Array.isArray(response.output) && response.output.length){
      const first = response.output[0];
      if(first.content && Array.isArray(first.content)){
        raw = first.content.map(c=>c.text || '').join('\n');
      } else if(typeof response.output_text === 'string'){
        raw = response.output_text;
      }
    }
    if(!raw && typeof response === 'string') raw = response;

    let parsed = null;
    try{
      const m = raw.match(/{[\s\S]*}/);
      parsed = m ? JSON.parse(m[0]) : null;
    }catch(e){
      parsed = null;
    }
    const result = parsed || { keywords: [], mood: '沉思、温和', summary: (raw || '我读到了你的梦，感受里有很多未完的情绪。')};
    return res.status(200).json({ analysis: result });
  }catch(err){
    console.error('analyze error', err);
    return res.status(500).json({ error: 'Analysis failed' });
  }
}
