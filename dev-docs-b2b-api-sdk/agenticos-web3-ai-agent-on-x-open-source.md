# AgenticOS: Web3 AI Agent on X (Open-Source)

## AgenticOS: Web3 AI Agent on X (Open-Source)

{% hint style="info" %}
**Quick Links:**\
**-** Demo: [https://x.com/ChainGPTAI](https://x.com/ChainGPTAI)\
\- Github: [https://github.com/ChainGPT-org/AgenticOS](https://github.com/ChainGPT-org/AgenticOS)&#x20;
{% endhint %}

***

### Introduction

AgenticOS is an open-source AI agent that autonomously generates and posts tweets on X (formerly Twitter) – purpose-built for the Web3 ecosystem. Built by ChainGPT using TypeScript and the ultra-fast Bun runtime, AgenticOS lets you effortlessly deploy an intelligent Twitter bot to enhance your online presence with 24/7 AI-driven insights . It leverages ChainGPT’s advanced Web3 AI model to produce content, helping crypto projects, developers, and influencers automate tasks like real-time market research, breaking crypto news, token analysis, and community engagement . By integrating directly with the ChainGPT API and Twitter API, AgenticOS serves as a tireless social media manager that keeps your followers informed and engaged around the clock.

***

### **Key Features and Benefits:**

* **AI-Powered Tweet Generation:** Uses ChainGPT’s Web3 LLM to create informative and relevant tweets on demand . Each generated tweet consumes 1 ChainGPT credit .
* **Scheduled Tweets:** Automatically post on a configurable schedule via Cron jobs, so you can share updates consistently (e.g. market updates every hour) .
* **Real-Time News via Webhooks:** Integrates with ChainGPT’s live news feed – subscribe to crypto news categories and auto-tweet breaking news as it happens .
* **Secure Token Management:** Stores Twitter OAuth2 tokens securely with encryption and auto-refreshes tokens as needed , ensuring long-term, hands-off operation.
* **Fast and Efficient:** Runs on the Bun runtime for high performance and uses TypeScript for reliability and developer-friendly code .

**Use Cases:** AgenticOS is ideal for Web3 startups, token projects, and content creators who want to leverage AI for social media.

**For example, you can deploy a Twitter bot to:**

* Post real-time crypto market updates and price trend analysis every few hours without manual effort.
* Share breaking news in DeFi, NFTs, or blockchain by auto-tweeting stories from ChainGPT’s AI-generated news feed moments after they drop.
* Provide daily educational threads or token analysis (e.g. morning recap of your project’s ecosystem or technical explainers) to keep your community informed.
* Boost community engagement by maintaining an active Twitter presence 24/7 – the AI agent can continuously share insights, polls, or replies (as configured) even when you’re offline.

**Live Demo:** The official [ChainGPT AI on X](https://x.com/ChainGPTAI) account is powered by AgenticOS – check it out to see the agent in action posting real-time Web3 insights and updates.

{% hint style="info" %}
_AgenticOS is part of ChainGPT’s open-source AI Agents initiative, helping bridge Web3 content with AI automation._
{% endhint %}

***

### Quick-Start Guide

Get your AgenticOS Twitter bot up and running in minutes. This quick-start will guide you through installation, setup, and your first tweet.

#### Prerequisites

**Before you begin, make sure you have the following:**

* **Bun Runtime (v1.0 or newer) –** Bun is required to run AgenticOS . Install it from [bun.sh](https://bun.sh) if you haven’t already.
* **Twitter API Credentials –** Create a Twitter app (with OAuth 2.0) via the Twitter Developer Portal to obtain a Client ID and Client Secret . These allow AgenticOS to post tweets to your account.
* **ChainGPT API Key –** Sign up on the ChainGPT platform to get an API key (from the ChainGPT API Dashboard) and ensure you have sufficient ChainGPT credits (each tweet uses 1 credit).
* **Node.js/NPM (optional) –** Not needed for running the agent (since Bun includes its own package manager), but having Node can be handy for other dev tasks.

#### Step 1: Clone the Repository and Install Dependencies

First, download the AgenticOS code from GitHub and install its dependencies:

```
# Clone the AgenticOS repository
git clone https://github.com/ChainGPT-org/AgenticOS.git
cd AgenticOS

# Install Bun (if not already installed)
curl -fsSL https://bun.sh/install | bash

# Install project dependencies using Bun
bun install
```

This will pull the latest AgenticOS source code and set up all required packages. The project is written in TypeScript, but Bun will handle transpiling and running it seamlessly.

#### Step 2: Configure Environment Variables

AgenticOS uses a .env file for configuration. Start by copying the example file and then open it in a text editor:

```
cp .env.example .env
```

**Fill in the required variables in your new .env file:**

* PORT: The port number for the AgenticOS web server (default 8000 is fine for local use).
* NODE\_ENV: Set to development for local testing (or production when deploying live).
* TWITTER\_CLIENT\_ID: Your Twitter app’s Client ID (from the developer portal).
* TWITTER\_CLIENT\_SECRET: Your Twitter Client Secret.
* ENCRYPTION\_KEY / ENCRYPTION\_SALT / ENCRYPTION\_IV: Random values used to encrypt sensitive data (32-char key, and salt/IV as hex). Generate secure random values for these.
* CHAINGPT\_API\_KEY: Your ChainGPT API key (from your ChainGPT account)&#x20;

**For example, your .env might look like:**

```
PORT=8000
NODE_ENV=development

TWITTER_CLIENT_ID=abc123yourID
TWITTER_CLIENT_SECRET=def456yourSecret

ENCRYPTION_KEY=your32charrandomstringhere!!!
ENCRYPTION_SALT=yourhexsaltvalue
ENCRYPTION_IV=yourhexivvalue

CHAINGPT_API_KEY=sk-xxxxxxxxxxxxxxxx
```

Ensure all required values are provided. These settings tell AgenticOS how to connect to Twitter and ChainGPT, and how to securely store tokens.

#### Step 3: Run AgenticOS in Development Mode

With configuration in place, you’re ready to start the AgenticOS server and authenticate your Twitter account:

```
# Start the development server
bun dev
```

On first launch, AgenticOS will start a local web server (by default on port 8000). It will likely prompt you to authenticate with Twitter – open http://localhost:8000 in your browser to link the agent to your Twitter account. You’ll be asked to authorize the app (using the Client ID/Secret you provided). Upon success, AgenticOS obtains an OAuth2 token for your Twitter account and encrypts it using the key/salt you provided, storing it for ongoing use.

Once authenticated, the agent is live! In development mode, it will log verbose output to the console. You should see messages indicating scheduled jobs or webhook setup (depending on your configuration). By default, it may not tweet immediately until you configure a schedule or webhook (covered below). Keep the bun dev process running to let the agent operate continuously.

#### Step 4: (Optional) Run in Production Mode

When you’re ready to deploy AgenticOS in a production environment (e.g. a cloud server or VM), use Bun’s production build:

```
# Build the project for production
bun build

# Start the production server
bun start
```

This will compile the TypeScript project and run the optimized bundle. In production mode, ensure NODE\_ENV=production in your .env and consider running the process via a process manager or as a service for reliability. The server will function similarly to development mode but with optimizations and less logging.

Your AgenticOS bot can now run 24/7, automatically tweeting based on your configuration.

***

### Configuration Details

AgenticOS is highly configurable. Let’s explore how to customize schedules, webhooks, and other settings to tailor the AI agent to your needs.

#### Twitter OAuth Setup and Token Storage

After the initial OAuth2 authorization, AgenticOS securely stores your Twitter tokens. The Encryption Key, Salt, and IV in the env file are used to encrypt the refresh token before saving (so your Twitter credentials remain safe at rest). The agent will automatically refresh the Twitter access token using OAuth2 when it expires , so you don’t need to re-authenticate manually. Important: Keep your encryption values secret and safe – if you change them, any stored token can’t be decrypted and you’d need to re-authenticate.

#### Scheduled Tweeting (Cron Jobs)

One core feature of AgenticOS is the ability to post tweets on a fixed schedule. The schedule is defined in the data/schedule.json file. You can edit this JSON file to specify what time of day to tweet and an associated topic or prompt for that timeslot.

**Example schedule.json format:**

```
{
  "09:00": "Daily DeFi market recap",
  "15:30": "Mid-day NFT trending collections update",
  "20:00": "Evening crypto news digest"
}
```

Each key is a time in 24h format (UTC), and each value is a brief prompt or topic for the tweet at that time. In the above example, the agent will attempt to post three times a day: at 09:00, 15:30, and 20:00 UTC. At each trigger, AgenticOS sends the prompt (e.g. _“Daily DeFi market recap”_) to the ChainGPT API, which returns an AI-generated tweet text, and then the agent posts that tweet to Twitter automatically . You can add or remove entries to adjust the schedule or change the content themes to suit your audience (e.g. weekly highlights, project-specific announcements, etc.).

{% hint style="info" %}
Tip: Start with a small set of times and prompts, then expand once you see the quality of tweets. Remember that each tweet uses one ChainGPT credit , so ensure your ChainGPT account has credits for the scheduled volume.
{% endhint %}

#### Real-Time News via ChainGPT Webhooks

AgenticOS can act as a live news broadcaster by integrating with ChainGPT’s AI-generated news feed. This allows your agent to tweet breaking news moments after it’s published. The workflow involves subscribing to news categories on ChainGPT and registering your agent as a webhook endpoint:

1. Subscribe to News Categories: Decide which categories of news you want (for example, “Breaking News”, “DeFi Updates”, “NFT Trends”, etc.). Use ChainGPT’s Web API to subscribe your agent to those categories. You can retrieve available categories via a GET request: GET https://webapi.chaingpt.org/category-subscription/ and choose relevant categoryIds. Then subscribe by sending a request like:

```
POST https://webapi.chaingpt.org/category-subscription/subscribe 
Content-Type: application/json

{
  "categoryIds": [2, 3]
}
```

1. In this example, we subscribe to categories with IDs 2 and 3 (the actual categories corresponding to these IDs can be obtained from the GET call). You should replace the array with the IDs of categories you wish to follow.
2. Register the Webhook Endpoint: Next, inform ChainGPT where to send the news updates. AgenticOS includes a predefined webhook route to receive news. Register your agent’s URL so ChainGPT knows where to post. For example, if your agent is running at https://myserver.com you would call:

```
POST https://webapi.chaingpt.org/category-subscription/register 
Content-Type: application/json

{
  "url": "https://myserver.com/api/webhook/"
}
```

2. This tells ChainGPT to send news events to the /api/webhook/ endpoint of your AgenticOS server . (When running locally for testing, you could use a tool like ngrok to expose your local server to receive webhooks.)

Once set up, ChainGPT will push news content to your agent whenever there’s a new story in the subscribed categories. AgenticOS will automatically receive the HTTP POST and convert it into a tweet – typically using the headline or summary provided in the news payload – then publish it on your Twitter account instantly . This is a powerful way to keep your followers up-to-date with the latest news without any manual intervention.

**Note:** Configure only the categories that are relevant to your audience to avoid spamming your feed. You can always unsubscribe or add new categories by calling the respective APIs again.

#### Integrating with ChainGPT’s AI (Prompt Customization)

Under the hood, AgenticOS calls ChainGPT’s Large Language Model API to generate tweet text. By default, it takes the prompt from your schedule (or the content of a news update) and asks the AI to produce a concise tweet. The prompts are designed to yield informative yet brief content suitable for Twitter’s format. You can customize this behavior in the source code if needed – for instance, to add hashtags, adjust tone, or append a call-to-action to each tweet. Look into the src/services/ or src/controllers/ directory in the project to find where the ChainGPT API is invoked and modify the prompt or parameters. For most users, the default prompt logic will suffice, but advanced developers have full control to tailor the AI’s output style to match their brand voice.

#### Adjusting Tweet Frequency and Content

**Every aspect of the tweeting logic can be adjusted to fit your needs:**

* **Tweet Timing:** Modify data/schedule.json to change when tweets go out. You can schedule as often as desired (even multiple times per hour) or as rarely as a few times per week. Times are in UTC, so convert from your local time accordingly.
* **Tweet Topics/Prompts:** Edit the text in schedule.json values. These act as the context for the AI. For example, set a prompt like "Update on $MYTOKEN development progress" to have the AI agent regularly tweet about your specific token or project.
* **News Handling:** By default, news webhooks are immediately turned into tweets. If you want to filter or tweak these, you can adjust the webhook controller. For instance, you might prepend something like “Breaking News:” to each tweet or skip certain types of news. This requires editing the code in src/routes/webhook.ts (or related) to customize how incoming news data is transformed into a tweet.
* **ChainGPT Model Parameters:** AgenticOS likely uses default settings for the ChainGPT API calls. If the API allows parameters (like temperature or max tokens for the response), you could adjust those in code to change the creativity or length of the tweets.

Through these customizations, you can tailor AgenticOS to your audience or community – whether that’s focusing on a particular blockchain’s news, using a certain tone (professional vs. casual), or integrating your project’s specific data into the tweets.

***

### Architecture & Workflow

To better understand how the pieces fit together, here’s an overview of AgenticOS’s architecture and tweet automation workflows:

<figure><img src="../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

_AgenticOS architecture and data flow: the agent runs as a Bun server, scheduling AI-generated tweets and handling incoming news events via webhook. Scheduled triggers and news updates both result in Twitter posts through the Twitter API._

At a high level, AgenticOS consists of a Bun-based web server with scheduled jobs and webhook routes:

* **Bun Server & Hono Framework:** AgenticOS uses the Hono web framework (running on Bun) to define HTTP routes for things like the Twitter OAuth callback and the ChainGPT news webhook. The server is what’s launched with bun dev/bun start and it keeps the agent running.
* **Scheduler (Cron Jobs):** A scheduler component reads the schedule.json and sets up cron-like jobs using Bun’s scheduling capabilities. At each scheduled time, the job triggers the workflow to generate and tweet content.
* **ChainGPT LLM API Integration:** When a scheduled job fires, the agent prepares a prompt (e.g. the topic from schedule.json) and sends a request to the ChainGPT API (authenticated with your API key) to generate a tweet. The response is expected to be a suggested tweet text, which the agent then proceeds to tweet.
* **Twitter API Integration:** The agent uses Twitter’s API (OAuth 2.0 with user context) to post tweets on your account. With the access token obtained and refreshed automatically, AgenticOS can call the Twitter API endpoint to create a tweet containing the AI-generated content.
* **Webhook for ChainGPT News:** The agent exposes an endpoint (e.g. /api/webhook/) that ChainGPT’s news service calls whenever there’s an update in your subscribed categories. Upon receiving a news payload, AgenticOS formats it (possibly truncating or adding hashtags as configured) and immediately posts it to Twitter via the same Twitter API integration.

**Workflow 1:** Scheduled Tweeting: Internally, at each scheduled time, AgenticOS triggers the sequence: (Cron trigger) → (generate content via ChainGPT) → (post to Twitter). This happens autonomously for each time slot you’ve defined .

**Workflow 2:** Live News Tweeting: When ChainGPT pushes a news update to the webhook, AgenticOS receives the HTTP request and directly uses that information to compose a tweet → then posts it to Twitter, almost in real-time . This means if, for example, a major crypto event happens and ChainGPT’s news service publishes an alert, your bot could tweet about it within seconds, keeping your followers informed faster than manual effort.

The diagram above illustrates these flows: scheduled events and incoming webhooks both converge on the AgenticOS core, which then utilizes ChainGPT’s AI for content and the Twitter API for publishing.

This architecture allows AgenticOS to run continuously and handle multiple input sources (timers or external triggers) in parallel, making your AI agent both proactive (scheduled tweets) and reactive (responding to news).

***

### Security Practices

Security is a priority in AgenticOS’s design, especially since it deals with API keys and access tokens. The following practices are implemented to protect your credentials and ensure stable operation:

* **Encrypted Token Storage:** Twitter OAuth tokens (particularly the refresh token that allows long-term access) are stored in an encrypted form. The encryption uses AES with your provided key/salt/IV from the .env file, meaning even if someone accesses the stored token data, it’s undecipherable without your encryption key. This keeps your Twitter account secure .
* **OAuth2 Token Refresh:** The agent handles the OAuth 2.0 flow for obtaining and refreshing tokens automatically. Once you’ve authorized the app, AgenticOS will use the refresh token to get new access tokens before the old one expires, in a secure and seamless manner. You don’t need to manually re-authenticate each time the token expires (typically every 1-2 hours), which is crucial for 24/7 operation.
* **Environment Variable Validation:** On startup, AgenticOS checks that all required configuration (env variables) are provided and valid. This prevents the agent from running with missing keys or misconfigured values. It fails fast with clear errors if something is amiss, helping you correct setup issues early.
* **Error Handling & Retries:** The code is written to gracefully handle API errors or network issues. For example, if the ChainGPT API call fails or Twitter is temporarily unreachable, AgenticOS can catch the error and retry or log it without crashing. This robustness ensures the agent remains up even if external services hiccup.
* **Least Privilege Principles:** The Twitter app used for AgenticOS only needs _write_ permissions to tweet (and read for basic profile info). No direct message or other intrusive permissions are required. Similarly, your ChainGPT API key scope is limited to content generation. Always keep your API keys private and limit their use to this agent to minimize risk.
* **Open-Source Transparency:** Because AgenticOS is open-source, its security can be audited by anyone. Developers are encouraged to review the code, suggest improvements, and trust that there are no hidden data collections or malicious behaviors.

By following these practices and using the recommended configurations, you can confidently run AgenticOS knowing your credentials and data are well-protected . (It’s still good policy to rotate your API keys periodically and revoke the Twitter app’s access if you ever stop using the agent.)

***

### Customizing AgenticOS for Your Needs

Every crypto community or project has unique needs – AgenticOS is flexible to accommodate those. Here are some ways you can customize your AI agent to make it truly yours:

* **Tailor the Tweet Content:** Adjust the prompts in schedule.json to focus on topics your audience cares about. For example, a DAO might schedule tweets about governance proposals or community calls, whereas a DeFi project might post daily TVL and yield updates. You can also modify the phrasing to match your brand voice (e.g. more formal vs. playful tone). The AI will take cues from the prompt text you provide.
* **Branding and Hashtags:** Since the agent is tweeting from your account, you might want to include your project’s hashtag or ticker symbol in every tweet. You could incorporate this into each prompt (e.g. _“Latest update on XYZ platform #XYZ”_), or alter the code in the tweet-posting function to automatically append a signature hashtag. This ensures every AI-generated tweet still carries your brand identity.
* **Frequency of Posts:** Customize how often the agent posts by editing the schedule. During critical periods (launches, events), you might increase frequency; during quiet times, you might scale back to avoid overwhelming followers. AgenticOS’s cron schedule is entirely under your control.
* **Selecting News Categories:** If using the news webhook feature, pick categories that align with your community. For instance, an NFT project might subscribe only to NFT-related news, while a blockchain platform might focus on general crypto news and technical developments. You can also run multiple agents for different accounts with different focuses, all using the AgenticOS framework.
* **Webhook Behaviors:** By default, every incoming news item triggers a tweet. In the code, you could add logic to filter or modify this. For example, maybe you want to batch multiple news items into a single tweet thread if they arrive in a short window, or ignore news that doesn’t mention certain keywords. Since you have the source, you are free to implement such rules to make the agent smarter about what it tweets.
* **Extend Functionality:** AgenticOS can serve as a foundation for more complex Twitter agents. You could extend it to reply to users, answer questions (using ChainGPT’s model to power a Q\&A feature on Twitter), or even integrate with other data sources (like pulling on-chain data or prices and tweeting alerts based on that). The open-source codebase is modular – for example, you can add new routes in the Hono server for custom APIs or new scheduled jobs in the jobs folder. Feel free to experiment!

Remember, any customization might require testing. Run your modified agent in development mode and verify it behaves as expected (tweets content on schedule, formats things correctly, etc.) before deploying changes to production. With some creativity, AgenticOS can become a personalized AI community manager for your specific niche in the crypto world.

***

### Contributing to the Project

AgenticOS is an open-source project by the ChainGPT team, and we welcome contributions from the community. If you have ideas for improvements, bug fixes, or new features, please contribute!

**To contribute:**

1. Fork the Repository on GitHub to your own account.
2. Create a Feature Branch – e.g. git checkout -b feature/my-improvement.
3. Commit Your Changes with clear messages describing the update.
4. Push the Branch to your GitHub fork.
5. Open a Pull Request on the main AgenticOS repository, explaining your changes and why they’d be beneficial.

Our team will review pull requests, provide feedback, and merge contributions that align with the project’s goals. Whether it’s improving documentation, refining the AI prompts, enhancing security, or expanding functionality, all constructive contributions are appreciated. Please ensure your code follows the existing style and includes any necessary tests or documentation updates.

If you encounter any issues or have questions, you can also open an issue on GitHub. This is the best way to report bugs or request features. We track GitHub issues for troubleshooting and future improvements .

***

### Support and Resources

* **GitHub Repository:** For source code, issue tracking, and project roadmap, visit the [AgenticOS GitHub page](https://github.com/ChainGPT-org/AgenticOS). Be sure to star the project if you find it useful!
* **Developer Documentation:** You’re currently reading the ChainGPT Developer Docs page for AgenticOS. For general ChainGPT API docs or other AI agents, refer to the [ChainGPT Docs site](agenticos-web3-ai-agent-on-x-open-source.md).
* **Community & Discussion:** Join the [ChainGPT Discord](https://discord.gg/FS2xpY28) (dev-chat channel) or follow [@Chain\_GPT on X](https://x.com/Chain_GPT) for announcements. Fellow developers and the ChainGPT team can help answer questions.
* **Live Demo (Twitter):** Follow [ChainGPT AI on X](https://x.com/ChainGPTAI) to see a live deployment of AgenticOS. This can give you inspiration for how the tweets look and confirm the agent’s capabilities in real-world usage.
* **ChainGPT API Access:** Manage your API key, check your credit balance, or purchase more credits on the [ChainGPT API Dashboard](https://app.chaingpt.org). Remember that a running AgenticOS will consume credits as it generates tweets , so keep an eye on usage or set up auto top-ups if available.

***

### Conclusion

AgenticOS empowers you to harness AI for social media automation in the crypto space. With a few simple steps, you can deploy a Twitter bot that never sleeps, providing valuable content to your followers and community. Whether it’s tweeting the latest market alpha, engaging users with insights, or instantly sharing breaking news, your AI agent can handle it – all while you focus on building your project or enjoying your day.

We’re excited to see what you do with AgenticOS. Happy coding, and welcome to the future of Web3 community engagement powered by AI!
