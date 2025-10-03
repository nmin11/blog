---
author: "Loko"
title: "Building a Discord RSS Feed Bot"
date: 2025-10-03
lastmod: 2025-10-03
description: "Developing a mini-project with Claude Code and Pulumi"
thumbnail: /thumbnail/feednyang.webp
toc: true
---

## Why Did I Create a Discord Bot?

Somewhat unexpectedly, several needs converged, and I decided to develop **"Feednyang"**, a bot that fetches tech blog feeds from various big tech companies. The needs were as follows:

1. I wanted to experience developing a web application with Ktor + Exposed.
2. I wanted to try Claude Code for the first time.
3. I felt a strong need to consistently write technical blog posts.

While thinking about what to build, an acquaintance from a bootcamp mentioned he was creating a Discord bot, which inspired me to try it as well. The manageable development scope, Discord bot as a new development area I'd never explored, and the opportunity to experience real users led me to develop Feednyang.

## What Is the Appropriate Architecture?

However, right from the start, I decided to exclude the Ktor + Exposed combination. A **serverless architecture** that invokes Lambda functions through AWS EventBridge scheduling was more efficient in terms of cost reduction and made sense since the logic didn't need to be always running. Looking at it from a serverless perspective, JVM-based languages' Lambda cold start issues felt burdensome. So I gave up on the Ktor experience and decided to create a simple function without a server framework.

To summarize from the conclusion, rather than the tech stack I simply wanted to experience, I prioritized the most suitable **appropriate architecture** for developing a tech blog feed bot. I thought that even for a lightweight project, developing the skill to select the right technology would have a positive influence going forward.

I chose **Go** as the language for implementing Lambda. While TypeScript also has decent performance and I have development experience with it, I've been avoiding it recently due to the need for bundling and transpiling processes. So I decided to proceed with development in Go, which is lightweight and has fast performance.

I also adopted **MongoDB** to check the list of feeds registered to channels and to register information about the last sent feed to prevent duplicate transmissions. I briefly considered DynamoDB, but MongoDB's free M0 cluster was much more attractive.

I chose **Pulumi** as the IaC tool. AWS CDK, which I used at my previous company, felt tightly coupled with CloudFormation, so it felt ambiguous to consider it true IaC. So I boldly introduced an IaC tool I was using for the first time. After using it, being able to configure components like MongoDB clusters, unlike CDK which can only configure services within AWS, felt like a significant merit.

Finally, **Claude Code** - as experiencing an AI Agent for the first time was a major motivation for starting this simple project, it's an essential element. It definitely helped me adapt to using Pulumi for the first time and allowed me to naturally utilize libraries like `discordgo` without barriers.

<img src="/blog/feednyang-tech-stack.svg">

## Even Simple Apps Need a Design Process

Actually, I initially thought I could implement it simply with a scheduler + Lambda. However, I suddenly became ambitious about supporting commands. Rather than just unidirectionally sending a fixed feed list, I thought having commands to add and remove feeds and check the current feed list would give the bot significance as something interactive.

Since it came to this, I decided to at least draw an **architecture diagram** before proceeding. I realized through this opportunity that even when making a simple project, writing a simple design plan helps organize thoughts and determine the development scope, which actually significantly shortens the time.

<img src="/blog/feednyang-architecture-diagram.svg">

I had rashly introduced MongoDB, but it was actually my first time using it. So I decided to organize the **MongoDB schema** as well. How should I design a schema for using document-based NoSQL? Well, I'm not sure yet. I designed it in a way that collects feed information by Discord channel, like designing JSON, but if I get to use MongoDB deeply later, I should study it more then.

```js
{
	"_id": ObjectId("discordChannelId"),
	"feeds": [
		{
			"blogName": "Naver D2",
			"rssUrl": "https://d2.naver.com/d2.atom",
			"addedAt": ISODate("2024-12-30T10:00:00Z"),
			"lastSentTime": ISODate("2024-12-30T10:00:00Z"),
			"lastPostLink": "FE News 25년 9월 소식을 전해드립니다!",
			"totalPostsSent": 100
		}
	],
	"createdAt": ISODate("2024-12-30T10:00:00Z"),
	"updatedAt": ISODate("2024-12-30T10:00:00Z")
}
```

## Can These Be Called Troubleshooting?

As mentioned earlier, I could configure a MongoDB cluster with Pulumi, but the initial setup took about 3 minutes, and there was an issue that M0 clusters cannot be updated. Therefore, the Pulumi infrastructure configuration was actually conducted mainly around AWS. Nevertheless, I could feel overwhelming convenience compared to CDK in managing secret values.

The most regrettable part of this project is that I left MongoDB's IP restrictions open. To set IP restrictions, NAT Gateway and VPC connection are needed. However, since the cost would increase by about 100 times, I decided not to proceed with it in the current project. This is also a point to leave for future challenges if I use MongoDB again.

Among the internal logic, the duplicate feed checking logic was quite tricky. During the initial Discord channel registration, I gave the wrong `lastSentTime` value and experienced message spam. I resolved this by setting the transmission time and URL based on the latest post of that feed when registering a new feed. Another problem was that when a feed posted 2 or more items within one scheduling period, the same feeds would appear in the next scheduling. This seemed to be because RSS files like `rss.xml` are displayed in reverse chronological order, passing through the time and URL-based duplicate check logic. So I added a flag value and modified it to update the last sent post information for the first item.

[Feednyang GitHub Repository](https://github.com/nmin11/feednyang)

## What I Learned

First, it felt rewarding as it's my first output created with an AI Agent. I should continue to actively utilize AI tools to maximize output going forward.

I also keenly felt that design capabilities are becoming more important than code writing capabilities thanks to AI. Seeing how I kept getting stuck on the design part even when making such a simple project, I realized that I lacked a lot of design experience.

So now I need to create a substantial side project in earnest. While I'd like to do a few more practice games like this one, I've become eager to hurry up and create my own side project with my name on it, something I've been dreaming about for a long time.

And let's make sure to write a blog post at least once a month.
