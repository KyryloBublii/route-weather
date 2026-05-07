---
title: Route Weather
year: 2024
languages: [Java]
tags: [Java, Spring Boot, Thymeleaf, openweathermap, Google Maps, Docker, Gradle, REST API]
status: complete
cover: photos/main.jpg
---

## What is it

A Spring Boot web application that shows real-time weather conditions at both ends of a driving route. The user enters two city names, sees the driving directions rendered on an embedded Google Map, and gets current temperature and weather description for each city — all on a single page.

## Problem it solves

When planning a road trip you typically check Google Maps for directions and a weather app for conditions separately. Route Weather combines both into one view: enter origin and destination, get the route and the weather at both points without switching tabs.

## How it works

- **Hardware:** No embedded hardware — runs as a web server on any JVM-capable machine or container.
- **Software:**
  - Java 17 + Spring Boot 3.3.0 (MVC controller, Thymeleaf template, reactive WebClient)
  - OpenWeatherMap REST API (`/data/2.5/weather`) for live weather data
  - Google Maps JavaScript API with Directions Service for route rendering
  - Temperature conversion from Kelvin (API default) to Celsius done server-side in `WeatherConverter`
  - In-memory per-IP rate limiter (`ConcurrentHashMap<String, AtomicInteger>`) capped at 5 requests per IP per day, reset nightly via `@Scheduled(cron = "0 0 0 * * ?")`
  - Docker multi-stage build: Gradle build stage → slim OpenJDK 17 runtime image, exposes port 8080
- **Key challenge:** Enforcing fair-use rate limiting without a database. `RequestTrackingService` uses `ConcurrentHashMap` with `AtomicInteger` counters keyed by IP address, making the check thread-safe and lock-free. A scheduled cron job clears all counters at midnight so limits reset daily without any external scheduler.

## Results

- Functional end-to-end: route rendering and weather fetched and displayed in a single request cycle
- Abuse protection in place: users hitting the 5-request daily cap receive a clear error message and are blocked until midnight
- Fully containerized: `docker build` produces a self-contained image ready to deploy anywhere

## Photos

No photos directory found in the repository at time of writing.
