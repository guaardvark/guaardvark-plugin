# Guaardvark for Claude

Guaardvark is a self-hosted AI studio: image, video, music, voice, agents and document search
running on your own computer and your own graphics card. This plugin lets Claude drive the
Guaardvark you have installed. You ask in plain words ("make a 5-second clip of a fox in the
snow", "what do my contracts say about renewals"), and Claude does the work through your copy
of Guaardvark. Images, video, music and voice are made on your own graphics card.

## What you need

- Guaardvark installed and started once, from
  [github.com/guaardvark/guaardvark](https://github.com/guaardvark/guaardvark). It runs on Linux
  with one NVIDIA card; video generation needs a 16 GB card.
- Claude Code, or a Cowork session that runs on the same computer.

The plugin does not work in claude.ai web chat. Web chat runs on Anthropic's servers and cannot
start a program on your computer, so the skills load there but cannot reach Guaardvark.

## Set it up

1. Add the plugin to Claude. In Claude Code you can also add it from the main Guaardvark
   repository: `/plugin marketplace add guaardvark/guaardvark`, then
   `/plugin install guaardvark@guaardvark`.
2. When it asks for **Guaardvark checkout**, give the folder that holds Guaardvark's `start.sh`.
3. Ask Claude to check the connection ("is Guaardvark running?"). The setup skill reports which
   features are available right now.

## What it can do

| Skill | What it does |
|---|---|
| setup | Connects to your Guaardvark and lists what is running |
| image | Creates and edits images, cut-outs, inpaint and outpaint, batches of prompts |
| video | Text-to-video, image-to-video, first and last frame animation, clips with their own sound |
| music-video | Turns a song into a video cut on the beat, with an approval step before rendering |
| film-crew | Turns an idea or a screenplay into a short film, with your approval at casting and storyboard |
| cast | Keeps the same face or object across pictures and videos, and trains a LoRA for it |
| music | Songs with vocals, instrumentals and sound effects |
| voice | Narration and text-to-speech; voice cloning only with the speaker's consent |
| upscale | Enlarges images and video to 4K or 8K |
| models | Adds an image or video model from a Hugging Face link |
| knowledge | Answers from your own indexed documents, with sources |
| code | Searches and maps code repositories Guaardvark has indexed |
| swarm | Runs several coding agents on one codebase, each in its own git worktree |
| ops | Graphics card memory, logs, stuck jobs, starting and stopping Guaardvark's services |
| outreach | Drafts social media replies for your review; it never posts on its own |

## What runs, and where your data goes

- The plugin starts one program: Guaardvark's MCP server, from the checkout you named, through
  `scripts/mcp_launcher.sh`. The skills also call Guaardvark directly on your computer, at
  `http://localhost:5000` by default or the address in `GUAARDVARK_URL`.
- The plugin itself sends nothing to any outside service and collects nothing.
- A few Guaardvark features reach the internet when you ask for them, under Guaardvark's own
  settings: web search and reading a web page, downloading a model from Hugging Face (only after
  you choose Install), syncing with your other machines through the Interconnector, and outreach
  drafts for Reddit, Discord, X and Facebook, which are posted only after you approve each one
  in Guaardvark's Studio.
- What you send to Claude in the conversation is handled by Claude under your Claude account's
  terms, as with any other conversation.

## License

MIT. See [LICENSE](LICENSE). More about Guaardvark: [guaardvark.com](https://guaardvark.com).
