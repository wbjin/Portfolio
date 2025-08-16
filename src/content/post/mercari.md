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
platform: translation, migration from legacy platform, cart, and operations tooling
for customer support agents. I also wrote a proposal for self hosting LLMs
instead of relying on external services.

### Backend
- Translation: Some product details are translated using LLMs. I refactored the
translation module to support versioning of translations with different models.
- Data migration: I helped with migrating from data from a legacy platform to
the GOP platform. I wrote various complex SQL queries across different
databases and tables using Big Query.
- Cart: I wrote a new endpoint for getting the number of unique items in a
user's cart. I worked with frontend engineers for designing protobufs.
- Operations too: Most of my time was spent on helping build out the operations
tool for GOP. I worked on working with a different team at the company in
charge of authentication in integrating authentication using JWTs. I helped
write proposals for a new way of doing authorization (checking that a user has
permissions for a certain action). I also wrote desgin documents and
implemented various feature endpoints such as retreiveing order data.


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

### Self hosting LLMs
Along with my actual work on GOP, I took initiative and worked on a proposal to
replace external LLM services with an internally hosted one. By using a GCP VM
with a Nvidia L4 GPU and hosting LLMs with vLLM, I was able to propose a
solution that cut 25% in costs.
