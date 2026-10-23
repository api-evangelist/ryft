---
title: "Streaming with Apache Iceberg: The Operational Problems at Scale"
url: "https://www.ryft.io/blog/streaming-with-apache-iceberg-the-operational-problems-at-scale"
date: "2025-12-08"
feed_url: "https://www.ryft.io/blog/rss.xml"
---
Streaming into Iceberg creates three operational problems most teams don't see coming: small files pile up faster than you can compact them, storage costs climb because you're paying for data you've already replaced, and merges take much longer than they should. After seeing these problems in real production data lakes, we decided to share more about the causes and possible solutions.
