---
title: Vertex AI Gemini shared capacity and 429 RESOURCE_EXHAUSTED
scope: tools/vertex-ai
status: verified-with-limits
updated: 2026-10-21
---

# Vertex AI Gemini throughput and 429

Vertex AI Gemini pay-as-you-go can use Dynamic Shared Quota (DSQ): requests draw from a shared pool whose availability changes with demand. When that pool lacks capacity, the request can be rejected with HTTP 429 `RESOURCE_EXHAUSTED`, commonly `Resource exhausted, please try again later`. This is a serving-capacity failure, not evidence that Cloud Billing is disabled or prepaid balance is empty. Provisioned Throughput is a separate paid option for reserved capacity; do not purchase it just to get a practice batch through without the user's explicit decision.

Google's error guide also lists project quota exhaustion as another 429 cause, so inspect the exact error and quota state before attributing every 429 to shared capacity. For `Resource exhausted, please try again` on pay-as-you-go/DSQ, Google describes temporary shared-pool demand and recommends retrying with exponential backoff; avoid rapid repeated requests. A 429 may clear as capacity changes. See [Vertex AI inference API errors](https://cloud.google.com/vertex-ai/generative-ai/docs/model-reference/api-errors) and [throughput quota / DSQ](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/resources/throughput-quota).

Verified 2026-10-21 in the TOEFL sprint project: a global-endpoint Gemini audio request succeeded for Q01 on retry and Q08 on first attempt, while Q01's first request and Q02 returned HTTP 429 `RESOURCE_EXHAUSTED` with `Resource exhausted. Please try again later.` A quota-info CLI query did not complete, so this run could not distinguish project quota from shared-pool contention conclusively. The safe response was to stop the batch and preserve successful results, rather than hammer the endpoint. Audio upload/response success proves the path works, not that generated pronunciation or TOEFL scores are calibrated.
