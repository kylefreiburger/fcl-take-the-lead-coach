[README.md](https://github.com/user-attachments/files/29988157/README.md)
# Take the Lead Coach — deployment guide

This is a small two piece app.

- `public/index.html` is the chat page members see. It is plain HTML, CSS, and JavaScript, no build step.
- `netlify/functions/chat.js` is a serverless function that holds your Anthropic API key and the coaching system prompt, and forwards each conversation to Claude. The browser never sees your API key.

## 1. Get an Anthropic API key

Go to console.anthropic.com, create or open your organization, and generate an API key under API Keys. Copy it somewhere safe, you will paste it into Netlify in step 3.

## 2. Deploy to Netlify

The easiest reliable path is connecting a GitHub repo, because Netlify then builds the function correctly every time.

1. Create a new GitHub repo and push this whole folder to it (`public/`, `netlify/`, `netlify.toml`, `README.md`).
2. In Netlify, click **Add new site > Import an existing project**, and connect that repo.
3. Netlify will read `netlify.toml` automatically. Publish directory is `public`, functions directory is `netlify/functions`. You do not need to change the build command, there isn't one.
4. Click deploy. It will finish quickly since there is nothing to compile.

If you would rather not use GitHub yet, you can deploy from your computer with the Netlify CLI (`npm install -g netlify-cli`, then `netlify deploy --prod` from inside this folder). Drag and drop deploys in the Netlify web UI do not run functions, so use the CLI or GitHub, not the drag and drop box.

## 3. Add your environment variables

In Netlify, go to **Site configuration > Environment variables** and add:

- `ANTHROPIC_API_KEY`, required. Paste the key from step 1.
- `ACCESS_CODE`, optional but recommended. Pick a short code, something like `TAKETHELEAD26`. Share this with your academy members. Anyone without it cannot use the coach, which keeps random internet traffic off your API bill.

After adding variables, trigger a redeploy (Netlify does this automatically on save in most cases, otherwise use **Deploys > Trigger deploy**).

If you set `ACCESS_CODE`, no other change is needed, the page already asks for a code before starting. If you decide not to use a code at all, open `public/index.html`, find the line `var ACCESS_GATE_ENABLED = true;` near the top of the script, and change it to `false`. Leave `ACCESS_CODE` unset on the server in that case too.

## 4. Test it

Open the live URL Netlify gives you (something like `https://your-site-name.netlify.app`). Enter your access code if you set one, and the coach should open with itself, asking how hard to push and which of the three signals fired. Try a full rep end to end before sharing the link.

## 5. Put it in front of members

Mighty Networks supports embedding external pages. The simplest route is a custom link in your Academy space pointing straight at the Netlify URL, opening in a new tab. If you want it to feel native inside a Mighty Networks page, most content blocks there support an embed or iframe option, in which case use:

```html
<iframe src="https://your-site-name.netlify.app" style="width:100%; height:640px; border:0; border-radius:12px;"></iframe>
```

Test the iframe version specifically on mobile, since some embed contexts restrict height.

## Customizing

**The coaching protocol itself** lives at the top of `netlify/functions/chat.js` in the `SYSTEM_PROMPT` constant. Edit it there directly, it is the same text as your starter prompt document. Redeploy after any change.

**The model.** The function currently calls `claude-sonnet-5`. That is a strong, well rounded choice for this kind of coaching conversation. If you run high volume and want to reduce cost, `claude-haiku-4-5-20251001` is faster and cheaper and still follows instructions well, change the `MODEL` constant in `chat.js` to test it.

**Response length.** `MAX_TOKENS` is set to 700, generous headroom for the 60 to 150 word responses the protocol asks for, plus the longer opening message. You can lower it if you want to hard cap cost per turn, but do not set it so low that a response gets cut off mid sentence.

**Colors, type, copy.** Everything visual is in `public/index.html`, in the `<style>` block at the top and the markup below it. Brand colors are defined once as CSS variables (`--navy`, `--blue`, `--horizon`, and so on) near the top of the stylesheet, change them there and they update everywhere.

## A note on the access code

The code check is a simple filter, not real authentication. It stops casual traffic and search bots from hitting your API key, but someone determined could still read the code out of the page's network requests and share it. For a closed academy audience this is normally enough. If you want real per member accounts later, that is a bigger project involving Mighty Networks' own membership data or a login system in front of the app, worth a separate conversation when you get there.

## Cost awareness

Every message a member sends triggers one API call, and every call resends the whole conversation so far, so cost per rep grows a little as a conversation gets longer, the same way any chat based AI product does. There is no volume cap built in beyond the access code. If this gets real usage, keep an eye on your Anthropic console usage page for the first couple of weeks so you know what a typical rep costs before it is in front of hundreds of members.
