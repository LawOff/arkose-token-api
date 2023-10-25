# Research on ChatGPT-4's Arkose Token Mechanism

This repository represents my ongoing investigation and understanding of how ChatGPT-4 utilizes the Arkose token system for its security measures. Through my research, I've crafted some preliminary tools and scripts to interact with this mechanism.
Everything you'll find here is largely inspired by a lot of information I've found, and my own deducations, so think of it as a wiki, and feel free to justify me, add more information or anything else appropriate.

## Overview:

With a ChatGPT Pro subscription you have access to GPT-4, OpenAI has integrated an additional layer of security via Arkose Labs' bot detection solution. 
Unlike GPT-3.5-Turbo, GPT-4's requests necessitate a unique `arkose_token` for each interaction.

The crux of this research lies in understanding this token's dynamic behavior and how it integrates with GPT-4's request lifecycle.

## Preliminary Findings:

1. The `arkose_token` is required for every GPT-4 request.
2. This token is ephemeral and changes for each request.
3. Prior to the main GPT-4 request, the browser seems to initiate an Arkose token fetch.
4. Direct interactions with GPT-4 without this token seem to be rejected:
`Our systems have detected unusual activity from your system. Please try again later.`

## Prototype Deployment:

You can find a simple script on this repo to generate such a token, which is deployable via Vercel:

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2FLawOff%2Farkose-token-api)

## Preliminary Implementation:

Here's a script using Puppeteer that attempts to fetch the Arkose token from a deployed instance:

```javascript
const puppeteer = require('puppeteer');

async function fetchTokenFromSite() {
    const browser = await puppeteer.launch();
    const page = await browser.newPage();

    await page.goto('https://YOURWEBSITE.vercel.app/');

    // Wait for the token attribute to be present
    await page.waitForSelector('body[data-token]');

    // Extract the token
    const token = await page.evaluate(() => {
        return document.body.getAttribute('data-token');
    });

    await browser.close();

    return token;
}

// Testing the function
fetchTokenFromSite().then(token => {
    console.log('Token:', token);
}).catch(err => {
    console.error('Error:', err);
});
```

## Example Payload with Arkose Token:
```javascript
let tokenArkose = fetchTokenFromSite();
```
```javascript
const payload = {
    action: "next",
    messages: [
        {
            id: conversationID,
            author: {
                role: "user"
            },
            content: {
                content_type: "multimodal_text",
                parts: ["QUESTION TO CHATGPT HERE"]
            },
            metadata: {}
        }
    ],
    parent_message_id: parentID,
    model: "gpt-4",
    history_and_training_disabled: false,
    arkose_token: arkoseToken,
    conversation_mode: {
        kind: "primary_assistant"
    },
    force_paragen: false
};
```


## Inspirations:
[linweiyuan - chatgpt-arkose-token-api](https://github.com/linweiyuan/chatgpt-arkose-token-api)

[flyingpot - funcaptcha](https://github.com/flyingpot/funcaptcha)

## Contributions:
This is a dynamic research space, and I invite community members to contribute their insights, findings, or potential solutions.
If you have relevant knowledge or techniques, please raise an issue or submit a PR. Together, we can better understand this intriguing aspect of ChatGPT-4.

### This README is crafted to reflect a research-focused tone while detailing your exploration and experiments on the Arkose token mechanism with ChatGPT-4.
