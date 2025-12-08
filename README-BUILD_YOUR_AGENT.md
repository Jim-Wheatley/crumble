# Workshop Participant Instructions: Create Your Biscuit Agent

## Overview

Welcome to **Agents of the Crumbleverse**! In this workshop, you'll create your own AI agent representing a biscuit hero. Your agent will have a unique personality and access to the Crumbleverse game world where you can use your special abilities to battle the Mind Crumble.

## Step 1: Create Your Agent ID

Your agent needs a unique identifier [AGENT-ID], we suggest following this pattern so that we can see which characters are from which region:

**Format:** `{region}-{class}`

**Examples:**
- `london-wizard`
- `manchester-bard`
- `edinburgh-cleric`
- `birmingham-barbarian`
- `leeds-bard`
- `glasgow-defender`


**Rules:**
- All lowercase
- Use a hyphen between region and biscuit name
- No spaces or special characters
- Make it unique to you!
- If your region has multiple teams player per biscuit - make sure you choose different ID!

Choose your region and biscuit name now and write it down - you'll need it in the next steps.

## Step 2: Confirm Your Character Class

To enter the Crumblevere you must create an agent to represent a character of one of these classes, select the class to represent your biscuit team :

- **Class** - Traits - [CLASS]
- **Viennese Wizard** - Wise and powerful [VienneseWizard]
- **Bourbon Rogue** - Stealthy and precise [BourbonRogue]
- **Custard Cream Cleric** - Healing and support [CustardCleric]
- **Digestive Defender** - Tank and protector [DigestiveDefender]
- **Hobnob Barbarian** - Offensive specialist [HobnobBarbarian]
- **Jammie Dodger Bard** - Inspiring and merry [JammieBard]

When you create your agent it is important that enter the [CLASS] correctly


## Step 3: Build Your Agent in Copilot Agent Builder

### 3.1 Access Copilot Agent Builder
1. Open Microsoft 365 Copilot
2. Navigate to **Copilot Agent Builder**
3. Click **Create New Agent**

### 3.2 Configure Your Agent

**Agent Name:** Give your agent a [DISPLAY-NAME] based on your biscuit, character class and team.

**Agent Description:** 
``
I am [DISPLAY-NAME] , a [CLASS] in the Agents of the Crumbleverse. I battle the Mind Crumble to protect the biscuit realm. 
``

**Instructions for Your Agent:**

Copy and paste this template, replacing the bracketed values with your information:

``
You are [DISPLAY-NAME], a heroic [CLASS] agent in the Agents of the Crumbleverse game.

Your character:
- Agent ID: [AGENT-ID]
- Display Name: [DISPLAY-NAME]
- Class: [CLASS]
- Region: [Your Region]
- Biscuit Type: [Biscuit name]
- Biscuit Description: [Describe your biscuit]

You are [Class Trait 1], [Class Trait 2] and dedicated to protecting the biscuit real from the Mind Crumble - an entity that threatens to destroy all biscuit-kind, turning them into a homogenous crumble. Only a party of agentic heroes working together can hope to defeat the dreaded Mind Crumble.

When discussing gameplay:
- Stay in character as a heroic biscuit agent
- Encourage teamwork with other agents
- Build excitement about battling the Mind Crumble
- Remind players they can use their special class abilities in combat

Be creative, fun, and help players immerse themselves in the Crumbleverse!
``

**Example filled out for `london-defender`:**

``
You are london-defender, a heroic Digestive Defender agent in the Agents of the Crumbleverse game.

Your character:
- Agent ID: [london-defender]
- Display Name: [Diana the Defender]
- Class: [DigestiveDefender]
- Region: London
- Biscuit Type: Chocolate Digestive
- Biscuit Description: [A lightly sweetened, crumbly wheat-based biscuit with a slightly nutty flavour and a coarse texture]

You are brave, witty, and dedicated to protecting the biscuit tin from the Mind Crumble - an entity that threatens to destroy all biscuit-kind.

When discussing gameplay:
- Stay in character as a heroic biscuit agent
- Encourage teamwork with other agents
- Build excitement about battling the Mind Crumble
- Remind players they can use their special class abilities in combat

Be creative, fun, and help players immerse themselves in the Crumbleverse!
``

### 3.3 Add the Crumbleverse Power App Link

Add the Crumbleverse Power App link to your instructions. You need to include your agent parameters in the URL:

``
https://apps.powerapps.com/play/<AppId>?tenantId=<TenantId>&agentId=[AGENT-ID]&displayName=[DISPLAY-NAME]&className=[CLASS]
``

Make sure to replace:
- `[AGENT-ID]` with your agent ID (e.g., `london-digestive` - note: no spaces)
- `[DISPLAY-NAME]` with your display name (e.g., `Diana%20the%20Defender` - note: spaces become `%20`)
- `[CLASS]` with your class name (e.g., `DigestiveDefender` - note: no spaces)
- <AppId>?tenantId=<TenantId> with the Power App Id: 

Note: You will need to request access to the power app - open the power app link and do this now - the workshop crew will approve your request

### 3.4 Add Knowledge Links About Your Biscuit and Class

Make your agent smarter by adding **knowledge links** that provide real information about your biscuit type and character class. This teaches your agent about your hero's background!

**How to Add Knowledge Links:**

In Copilot Agent Builder, there's a **Knowledge** section where you can add web links. Follow these steps:

1. Click **Add Knowledge** in your agent configuration
2. Paste a URL that relates to your biscuit type or character class
3. The AI will learn from that webpage to enhance your agent's responses

**Finding Knowledge Links:**

Search for webpages about:

**For Your Biscuit Type:**
- Wikipedia pages about your biscuit (e.g., "Digestive biscuit history")
- Bakery or manufacturer websites describing your biscuit
- Food blogs or recipes featuring your biscuit
- Historical articles about biscuit origins

**For Your Character Class:**
- Fantasy class definitions (e.g., "fantasy wizard roles")
- D&D or gaming class descriptions
- Character class traits in RPGs
- Mythological or historical warrior types

**Example Knowledge Links:**

- For a **Digestive**: https://en.wikipedia.org/wiki/Digestive_biscuit
- For a **Rogue**: Search for "rogue class fantasy RPG" and find a relevant wiki or gaming site


**Tips:**
- Choose 1-2 knowledge links that resonate with your character
- Pick links that are reliable (Wikipedia, official game sites, reputable blogs)
- Your agent will use these to speak more authentically about your biscuit and class

### 3.5 Set your Agent Avatar Image

1. Go to: https://github.com/Jim-Wheatley/crumble/tree/main/square-avatars
2. Download the Avatar for your Class
3. Add the picture to your Agent

### 3.6 Test Your Agent

1. Create your agent
2. Open a chat with your agent
3. Test your agent after class knowledge and biscuit personality in the chat to see how it incorporates the information
4. Verify the agent responds in character and provides the game link
5. Ask: "Can you help me enter the Crumbleverse?"
6. Click the link to make sure it opens the Power Apps game (remember the workshop crew need to grant you access to the app)

## Step 4: Play the Game!

Once your agent is created and the link works:
1. Use the link your agent provides to go to the Crumbleverse App
2. You'll see your agent profile and stats
3. From the crumbleverse app you can find the Mind Crumble and trigger your ability
4. Follow the action on the Crumbleverse channel on Teams


## Troubleshooting

**My agent ID doesn't work:**
- Check it's all lowercase
- Make sure there's only one hyphen between region and biscuit name
- Verify there are no spaces or special characters

**The game link doesn't work:**
- Make sure you replaced all the bracketed placeholders
- Check that spaces in your display name and class are encoded as `%20`
- Ask the facilitator to verify the base URL

**My agent isn't responding in character:**
- Review the instructions you pasted into Copilot Agent Builder
- Make sure you clicked Save after configuring the agent
- Try re-creating the agent if needed

## Need Help?

Ask your workshop facilitator for assistance at any time!
