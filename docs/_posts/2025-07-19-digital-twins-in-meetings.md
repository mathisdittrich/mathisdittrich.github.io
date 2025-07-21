---
layout: post
title:  "I created a digital twin of myself that joins low-value meetings for me"
date:   2025-07-19 
---

## The rise of AI notetakers

Haven't we all been in these meetings that seem to go on forever without achieving anything in the end? Picture a mandatory team meeting with far too many participants, each delivering lengthy updates about stuff that's not remotely related to your work.

Given this frustration, it's hardly surprising that more and more people are sending AI notetakers to meetings, rather than showing up themselves. A recent [Washington Post article](https://www.washingtonpost.com/technology/2025/07/02/ai-note-takers-meetings-bots/) recounts scenes of people joining a meeting where these AI bots outnumbered the human participants. In some of these stories, the attendee was even the only human in a meeting *invaded* by AI notetakers. While this will undoubtedly remain a rare case, there is no denying that AI notetakers are rapidly becoming an unavoidable fixture in our daily working lives.

{% include youtubePlayer.html id="wB1X4o-MV6o" %}

## Why AI notetakers suck

This brings up new questions. For example, if you send an AI notetaker to a meeting in your place, is this equivalent to attending or skipping the meeting? 

Sure, the AI bots send their users a summary email after the meeting so that they should, in theory, get the necessary information. But do people actually read that email, or does it just end up unread in a crowded inbox? 

On the other hand, it's not unusual for people to join a meeting and never turn on their camera or microphone, probably playing some handy game on the side and not actually contributing anything to the meeting. But they can at least be called on to participate. An AI notetaker can't. *Let's fix that!*

## Why digital twins will be next 

For AI bots to truly represent a person absent from a meeting, they must actively participate by answering questions, providing TL;DR's, and offering suggestions, effectively becoming a *"digital twin"* of that person. This would mean integrating it deeply with the person's work tools (like Linear, Notion, GitHub, etc.), which isn't that hard these days, thanks to the hype around MCP servers. With that, your digital twin should theoretically already be equipped to deliver your stand-up in your next team meeting.

I found myself wondering: Is it already possible to create such a digital twin? *(Spoiler: The answer is yes!)*. But I wanted to go further: I aimed to build an *AI meeting agent* that could not only understand my work context, but also mimic my voice and communication style. Because why not? *So, let's dive in!*

## How to clone your voice

The first thing I needed was a convincing voice replica. I chose [ElevenLabs' Professional Voice Cloning](https://elevenlabs.io/docs/product-guides/voices/voice-cloning/professional-voice-cloning), which requires at least 30 minutes of high-quality audio. Luckily, I already had some recordings of myself from unreleased, scraped YouTube videos, which finally came in handy and saved me the trouble of making new recordings.

All I had to do was upload my audio, verify that my voice belongs to me *(a step I failed a couple of times)*, and wait for the fine-tuning process to finish. When that finally happened, I grabbed the `voice ID` from ElevenLab's dashboard, and my voice was ready for use through their API.

I was genuinely impressed by how well the model captured the unique features of my voice. I didn't even notice before that I make a very distinctive sharp "s" sound and long pauses between words. *But check out the video at the end of the article and hear for yourself!*

## This LLM has style 

The next step involved fine-tuning an LLM to respond in my tone and style. Fortunately, OpenAI's [Direct Preference Optimization (DPO)](https://platform.openai.com/docs/guides/direct-preference-optimization) method made this simple. It allows you to fine-tune models based on preferred and non-preferred example responses to specific prompts.

To create the necessary training dataset (about 100 examples formatted as a `JSONL` file are already enough), I *of course* avoided any manual work! Instead, I fed my YouTube transcripts to ChatGPT, letting it extract statements I made and generate prompts that would naturally lead to them, along with typical ChatGPT responses for comparison.

After preparing my dataset, I just needed to click myself through the web-based GUI of the OpenAI platform to start the fine-tuning process. I uploaded my `JSONL` file to the OpenAI platform, selected `gpt-4.1-2025-04-14`as the base model, and hit the "Create" Button. Soon after, my new personalized model was ready. The final step was to fetch the `Output model` name from the dashboard because it is the identifier we'll later use with the OpenAI API.

## Unleashing the digital twin

Finally, it was time to build and deploy the actual AI meeting agent. To ensure that it had all the necessary context for my meetings, the agent needed access to my work tools.

The easiest way to equip an AI agent with such skills is through the Model Context Protocol (MCP), which gives the agents a consistent way to connect to data and tools from external services. Almost all the popular applications you probably use in your daily work life, such as HubSpot, Linear, and Asana, have an MCP server up and running, with more being released every week.

So, now I only needed to worry about how I could get the agent into a video conference. Enter [joinly](https://github.com/joinly-ai/joinly), an open-source project I'm currently working on with two friends: It is a connector middleware specifically designed to enable AI agents to actively participate in video calls. How does it work? Ultimately, joinly is also just an MCP server that you can host yourself, providing your agent with essential meeting tools (such as `speak_text` and `send_chat_message`) alongside automatic real-time transcription.

Joinly already provides a `client_example.py` so that you don't need to write your agent from scratch. However, to make the agent understand that it is acting on my behalf, I had to tinker with its system prompt a bit. In the end, it came down to adding this line

> "You are Hannes-Bot, a meeting agent that impersonates the real employee Hannes and acts on their behalf in the meeting."

and describing my personality, especially how I act in meetings. Very hard to come up with something non-generic there. *Did you know that I am very proactive in meetings and try to set and work toward clear, predefined goals?* 

Lastly, you need to provide a `mcp_config.json` file, where you list the MCP servers your AI meeting agent should connect to. For the demo video you can see below, I just added the Linear MCP Server. 

Now, the time had finally come to send my digital twin to a meeting. All that was left was to spin up the joinly MCP server and specify my personalized LLM and voice replica.

```bash
uv run joinly --model-name <Output model> --model-provider openai --stt <STTProvider> --tts elevenlabs --tts-arg voice_id=<Voice ID>
```

Then I started the modified example client to connect to it and join the meeting

```bash
uv run examples/client_example.py <MeetingUrl> --config examples/mcp_config.json
```

Just like that, I was free from any unnecessary meetings and could spend my valuable time more productively *(and definitely not by scrolling through Reddit or reading HackerNews posts).*

Want to see it in action? Check out this entertaining demo video where I chat with my digital twin about productivity. *Turns out, it's impressively productive!*

{% include youtubePlayer.html id="2pnYkW4-Gbk" %}

## Final thoughts

I was amazed at how easy it was to create an interactive digital twin of myself that could attend meetings. On top, the approach was pretty straightforward and low-code. It mainly consisted of clicking through some dashboards and drinking coffee while waiting for a fine-tuning job to finish. So, almost everybody should be able to copy it and use their own digital twins to skip their low-value meetings. But should they do it? *Maybe not.* 

I will talk about the why in a second. But first, I want to discuss the implementation a bit: In hindsight, fine-tuning an entire personalized model seems very excessive. The same effect could probably be achieved by using few-shot prompting and providing the model with phrases you often use in meetings as context.

However, I would argue that it's questionable whether you really need your digital twin to mimic your voice and style anyway. It's pretty cool to be able to talk to it yourself, but it's probably just creepy for your colleagues and a waste on people who don't know you. Also, these fine-tuning methods are primarily available through closed-source vendors, forcing you to use them instead of the privacy-preserving alternative of using self-hosted LLMs and STT models.

Now, my thoughts on digital twins compared to AI notetakers: Having a digital twin in a meeting instead of an AI notetaker would definitely improve the experience for the organizer. For example, they could ask the bot what the employee who sent said bot is currently working on, or even request it to find that one specific value buried in a spreadsheet created by that employee yesterday. For the employee, this would mean fewer follow-up messages after the meeting.

Additionally, employees are more likely to see the Linear issue created by their digital twin during a meeting than to read about it in an AI notetaker's summary email. However, this could easily lead to even more follow-up questions because you completely lack context about how the issue was created.

This also opens up new problems that could completely call into question the value of decisions made in meetings attended by digital twins. Picture this: If my digital twin says in the meeting that I can do all 42 urgent bug fixes by tomorrow because my calendar is free, do I really have to? *Better not.*

In conclusion, digital twins seem useful for people who want to skip long, boring meetings where, for example, they only need to say one sentence that could easily be answered by looking something up on a specific Notion page. However, in meetings where lively discussions happen and actual decisions are made, digital twins may have the potential to disrupt the flow of the meeting even more than AI notetakers and cause a lot of chaos afterward.



