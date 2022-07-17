---
layout: post
title: First Go public PR merged
tags: [go, redfish]
---

While working on testing [hardware-event-proxy](https://github.com/jzding/hw-event-proxy) I used [Gofish](https://github.com/stmcginnis/gofish).
Now after my test was submitted, I wanted to give back and improve Gofish, so the events service will also work on other vendors like Dell and ZT systems, that do not follow the redfish spec on subscription and SubmitTestEvent.
It took some time, but finally [my code is merged](https://github.com/stmcginnis/gofish/pull/187), which is so cool.
I also learned a lot in the process, like the way Sean is unit testing his functions, yet I used the httptest way.