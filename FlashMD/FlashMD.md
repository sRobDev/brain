**What will I need?**

DB to store the markdown in - could really be anything, might as well use an easily built and scaled db like supabase or something. Maybe a hosted option? but let's go with supabase for now.

Supabase for storing markdown
Remix for frontend
vercel for hosting pointed to flashmd.app

**MVP Functionality**
- Home page w/ "Create New Paste" button
- Create New Paste links to `/new`
- New route lets you give a paste title and paste in the according markdown
- Default no deletion? Depends on how hard it would be to auto-delete after X amount of time 
- Upon submit, we hit an API route that does:
	- Create new UUID
	- Connect UUID to post (maybe just created by supabase?)
	- route to dynamic route `flashmd.app/<uuid>`
- Display formatted markdown w/ title
- No editing (yet)
- No user history or anything (yet)

**Remix setup**
- Vercel 
- Tailwind? - would give some content (how to use tailwind in Remix)
- Homepage (how to style this lol)
- `new.tsx` for route
- Styled form
- Submit action that hits the above detailed api 
- then goes to `$uuid.tsx` route - may need to experiment with this a bit, will remix let us have dynamic root routes or will they need to be under something like `flashmd.app/md/<uuid>` (this setup might honestly be easier regardless)
- Nicely styled page for the md files

# Project Work/SoC

### Adding Tailwind
- npm add tailwindcss
- mkdir `config`
- `touch config/tailwind.css`
- Then add a couple scripts to `package.json`:
```
"tailwind:dev": "tailwindcss --output ./app/styles/tailwind.css --config ./config/tailwind.js --watch",

"tailwind:production": "tailwindcss --output ./app/styles/tailwind.css --config ./config/tailwind.js --minify"
```

**Q: Do we have to have `tailwind:dev` running during dev? Or could we parallelize it with the npm run dev command?**

**A: You can parallelize it: `"dev": "npm run tailwind:dev & node -r dotenv/config node_modules/.bin/remix watch"` (or dev)**

