# Lynx API worker

Endpoints: `GET /api/health`, `GET /api/callouts`, `GET /api/channels`, `POST /api/waitlist {email}`,
`PUT /api/upload?kind=avatar|banner` (needs `Authorization: Bearer <Supabase access token>`, image body up to 2 MB; returns `{key,url}`), `GET /api/media/<key>`.

## Deploy
1. `cd lynx-worker && npx wrangler login`
2. Edit `ALLOWED_ORIGIN` in `wrangler.toml` to your site URL.
3. `npx wrangler deploy`
4. Test: `curl https://lynx-api.<your-subdomain>.workers.dev/api/health`

## Chat (client talks to Supabase directly)
```js
import { createClient } from '@supabase/supabase-js'
const sb = createClient(SUPABASE_URL, SUPABASE_PUBLISHABLE_KEY)

// send a DM (RLS checks you are the sender and not blocked; 20 msgs/min limit)
await sb.from('messages').insert({ sender_id: me, recipient_id: peer, body })

// live updates
sb.channel('dm').on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'messages' }, p => render(p.new)).subscribe()

// inbox, unread counts, mark read
const { data: inbox } = await sb.rpc('my_conversations')
const { data: unread } = await sb.rpc('my_channel_unreads')
await sb.rpc('mark_read', { p_peer: peer })
```
