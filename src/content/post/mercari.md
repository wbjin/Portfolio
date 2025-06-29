---
title: "Mercari"
description: "My project during my internship at Mercari"
publishDate: "August 2025"
tags: ["go", "work"]
---

## [Mercari](https://jp.mercari.com/)

Mercari is a Japan-based online marketplace that primary focuses on the C2C
business. Launched in 2013, it is one of Japan's fastest growing tech companies
and has expanded into the US, Taiwan, Hong Kong, and the fintech business within
Japan. The company has around 2000 employees with around 1500 of them being in
the Japan Business which was where I worked

## My work
I was on the Cross Border, Global One Product (XB GOP) team. The team is
responsible for building the cross border marketplace platform, allowing
Japanese users to sell items to users in Taiwan, Hong Kong, and eventually the
rest of the world. I focused specifically on the backend of three parts of the
platform: translation, migration from legacy platform, and operations tooling
for customer support agents.

### Translation
Content is translated using LLMs.

- Refactor translation module to support versioning of translations with
different models. Allows poor translations flagged by customers and support
agents to be retranslated with a different model
- Improve system prompt used for translation


### Migration from legacy platform
The cross border marketbaplace platform has two versions: Inhouse and GOP.
Inhouse was built very quickly as a way to prove that the cross border business
would be viable but becuase it is heavily based on the existing system that
powers the Japan marketplace, it is not scalable for widespread cross border
operations. GOP is the platform that was built specifically for cross border
operations and will replace Inhouse. I helped with the migration of data from
Inhouse to GOP

- Write cron job that logs orders and items in the orders that are in Inhouse
but missing in GOP
- Write SQL queries to validate order data that was migrated from Inhouse to
GOP, checking each field to make sure no data was lost or mapped incorrectly

### Operations tooling
Operations tooling for customer support agents was not implemented in GOP yet. I
was able to create and deploy a microservice for this new feature

- Create and deploy a microservice that authenticates customer support agents

## Microservices
Mercari strongly embraces microservices and modularization. The tech stack I
used was Go with gRPC on Google Cloud Platform. Through the internship, I became
really familiar with writing a new module/microservice, writing protobufs, and
deploying the microservice.

Some thoughts about microservice architecture: Separating a large application
into different module allowed the team I was on to ship different features
quickly and without down time. However, one thing I relaized was that it was
harder to read and understand what code was actually executed when a request
came into the application. Luckily, because GOP uses a monorepo with modules
that act as a microservice, it was still possible to find where things were in
the same repository. I couldn't imagine how hard it would be to find where
things are if modules that you interact with are in an entirely differnet
codebase altogether. This also meant that it was really hard to have a full
picture of what happens when a request comes through. Many full time engineers I
talked to knew the module that they were in charge of very well and in depth but
not as much about modules upstream or downstream. I think this can be a strength
and a weakeness. It allows engineers to specialize in one part of the
application and own that for themselves. They can be come really fast at solving
bugs, adding new features, and maintaining that moudle. However, I personally
feel like all engineers should have a baseline common understanding of how the
entire application works. I believe that it's important to have a shared
understanding when it comes to things like code style, error handling policy
(dieing quickly vs trying to recover), and the general state of the application
(is latency an issue? are we trying to add more features or stabilize existing
ones, etc etc)
