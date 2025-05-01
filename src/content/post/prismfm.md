---
title: "Prism.fm"
description: "My project during my internship at Prism.fm"
publishDate: "May 2024"
tags: ["typescrript", "php", "work"]
---

## [Prism.fm](https://prism.fm/about/)
Prism.fm is a startup in the music entertainment industry focused on creating software for booking
and managing concerts and events at venues.

## My role
I was a full stack engineer intern at Prism. The project I was placed on was building a leaderboard
and dashboard feature for top performing artists that are booked through Prism. The technologies
that I used were PHP with Laravel on the backend and Typescript with Angular on the frontend.

My main focus through the internship was enabling ranking and fetching of individual artist data. The
leaderboard and dashboard features were built on top of AWS's Open Search (Elastic Search), a
document based data store capable of fast data analytics and filtering. I built a datapipeline that
gathered individual artist and event data from the application's MySQL database and pushed that to
OpenSearch periodically. I also built a querying engine that abstracted the complicated JSON
generation needed for interfacing with OpenSearch.
