---
layout: post
title: Adding Seq logging to unity
categories: Seq Unity Logging
---

Seq is my logging server of choice, so as there were no existing integration with Seq, I wanted to see how far I could take an implementation. As unity doesn't have nuget support, I wanted an implentation that tied into unity's logging commands rather than something like serilog. The lack of unity support also limits the JSON serialization. The serializer included with unity is unable to serialize dictionaries or anonymous types, so can't have dynamic log properties.

This is just a proof of concept, but the limitations of unity json serialization and lack of formatted logging commands require a more extensive implementation. To provide a logging feature in line with what we're used to, it would require implementation of custom logging commands and either a customer JSON serializer, or dependency on one available in the asset store.

To use the script, just add it to a gameobject in the scene. It will automatically subscribe a callback to the unity logging.

{% gist c2dbeb103ae47ebde01341b22818afee %}