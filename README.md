# 8085 Lab

Static site. Three files, no build step, no dependencies, no backend.

    index.html         landing page
    lab-bench.html     assembler + simulator + 15 checked lab assignments
    trainer-kit.html   trainer-kit emulator (display + hex keypad)

## Deploy to Vercel

Option A - from this folder, no Git needed:

    npm i -g vercel
    vercel            # answer the prompts, accept the defaults
    vercel --prod     # promote it to your real URL

Option B - from GitHub:

    git init && git add . && git commit -m "8085 lab"
    git branch -M main
    git remote add origin https://github.com/<you>/8085-lab.git
    git push -u origin main

Then on vercel.com: Add New -> Project -> import the repo.
Framework Preset: Other. Build Command: leave empty. Output Directory: leave empty.

No vercel.json is needed - Vercel serves a folder containing index.html as-is.

## Anywhere else

The same folder works unchanged on Netlify (drag it onto app.netlify.com/drop),
GitHub Pages, Cloudflare Pages, or any static host. You can also just open
index.html from disk.

## Notes

- Fonts load from Google Fonts. Remove the two <link> tags in each file if you
  want it fully offline; the fallback stacks are already in place.
- Progress (solved assignments, your last program) is kept in the browser's
  localStorage, per device. Nothing is sent anywhere.
