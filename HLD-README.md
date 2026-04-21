# Uber SDE-2 HLD prep — how to use this folder

## Start here

1. Read **[HLD-UBER-SDE2-INTERVIEW-SPINE.md](./HLD-UBER-SDE2-INTERVIEW-SPINE.md)** once — it is the **canonical 14-beat map** (clarify → FR → NFR → scale → entities + APIs → architecture → deep dive → scaling → reliability → tradeoffs → observability & security → patterns → closing) and what a **bar raiser** listens for.
2. Open **one** problem guide (`NN-hld-*.md`). Right under the title you will see a **drive-order** blockquote — use it as a **checklist** while you talk.
3. Prefer **11** and **12** as the **style reference** for `#### Human interaction` density; use the **Cursor prompt** in the spine doc to thicken any guide that still feels thin.

## Strong Hire behaviors (compressed)

- **Clarify before boxes**; reflect back one sentence after scope answers.
- **Name defaults** (storage, index, consistency) and **when** you would change them.
- **Tie every pattern** to a component on your diagram.
- **Failure + degrade**: what the user sees when dependency X is slow or wrong.
- **Stop on time**: 60-second close, then silence.

## Problem guides (11–37)

| # | File |
|---|------|
| 11 | [11-hld-uber-eats-homepage.md](./11-hld-uber-eats-homepage.md) |
| 12 | [12-hld-ecommerce-product-browsing.md](./12-hld-ecommerce-product-browsing.md) |
| 13 | [13-hld-splitwise.md](./13-hld-splitwise.md) |
| 14 | [14-hld-whatsapp-web-messaging.md](./14-hld-whatsapp-web-messaging.md) |
| 15 | [15-hld-keyword-message-search.md](./15-hld-keyword-message-search.md) |
| 16 | [16-hld-notification-stock-alerts.md](./16-hld-notification-stock-alerts.md) |
| 17 | [17-hld-social-news-feed.md](./17-hld-social-news-feed.md) |
| 18 | [18-hld-uber-ride-sharing-backend.md](./18-hld-uber-ride-sharing-backend.md) |
| 19 | [19-hld-ride-matching-driver-dispatch.md](./19-hld-ride-matching-driver-dispatch.md) |
| 20 | [20-hld-real-time-driver-tracking.md](./20-hld-real-time-driver-tracking.md) |
| 21 | [21-hld-surge-pricing.md](./21-hld-surge-pricing.md) |
| 22 | [22-hld-eta-fare-estimation.md](./22-hld-eta-fare-estimation.md) |
| 23 | [23-hld-uber-eats-train-pnr-delivery.md](./23-hld-uber-eats-train-pnr-delivery.md) |
| 24 | [24-hld-food-delivery-order-dispatch.md](./24-hld-food-delivery-order-dispatch.md) |
| 25 | [25-hld-restaurant-recommendation-location.md](./25-hld-restaurant-recommendation-location.md) |
| 26 | [26-hld-restaurant-search-nearby.md](./26-hld-restaurant-search-nearby.md) |
| 27 | [27-hld-search-autocomplete.md](./27-hld-search-autocomplete.md) |
| 28 | [28-hld-rate-limiter.md](./28-hld-rate-limiter.md) |
| 29 | [29-hld-distributed-job-scheduler.md](./29-hld-distributed-job-scheduler.md) |
| 30 | [30-hld-logging-metrics-pipeline.md](./30-hld-logging-metrics-pipeline.md) |
| 31 | [31-hld-monitoring-alerting.md](./31-hld-monitoring-alerting.md) |
| 32 | [32-hld-distributed-cache.md](./32-hld-distributed-cache.md) |
| 33 | [33-hld-push-notification-service.md](./33-hld-push-notification-service.md) |
| 34 | [34-hld-chat-messaging-platform.md](./34-hld-chat-messaging-platform.md) |
| 35 | [35-hld-email-sms-notification.md](./35-hld-email-sms-notification.md) |
| 36 | [36-hld-url-shortener.md](./36-hld-url-shortener.md) |
| 37 | [37-hld-analytics-event-pipeline.md](./37-hld-analytics-event-pipeline.md) |

## Companion docs

- **[HLD-11-17-UNIFIED-INTERVIEW-SPINE.md](./HLD-11-17-UNIFIED-INTERVIEW-SPINE.md)** — same beats as the canonical spine; notes why **11–17** are good **style** anchors.
- **[HLD-11-17-ELEVATOR-PARALLEL-SPEC.md](./HLD-11-17-ELEVATOR-PARALLEL-SPEC.md)** — parallel study: how to rehearse **11–17** in one sitting without merging every file into one mega-doc.

Back to repo root: [README.md](./README.md).
