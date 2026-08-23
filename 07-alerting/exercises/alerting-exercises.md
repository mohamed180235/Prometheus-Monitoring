# Alerting Exercises

## Exercise 1 — High CPU

Create an alert when CPU exceeds 80% for 2 minutes.

## Exercise 2 — High Memory

Create an alert when memory exceeds 80% for 5 minutes.

## Exercise 3 — Target Down

Create a critical alert when any Prometheus target is down for 1 minute.

## Exercise 4 — cAdvisor Down

Create a critical alert specifically for the cAdvisor target.

## Exercise 5 — Alertmanager Routing

Route:

- `warning` alerts to a warning receiver
- `critical` alerts to a critical receiver

## Exercise 6 — Silence

Create a silence for `TargetDown` while intentionally stopping Node Exporter.

## Exercise 7 — Challenge

Create an alert for a container that consumes unusually high CPU.
