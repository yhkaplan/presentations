slidenumbers: true
slide-transition: true
theme: Work
autoscale: true

# Backend-Driven UI: Making Screens Dynamic
^ バックエンド駆動UI: 画面の流動化

---

# Self-intro
* Name: Joshua Kaplan (@yhkaplan)
* Work: minne @ GMO Pepabo
* Interests: 🤖CI/CD, 📦 frameworks, and more
* Hobbies: 🥯bread, 📚history, and 🏃running

---

# What is a backend driven UI?
* Extreme end: every individual view defined in JSON
* Less extreme: order of and type of view defined in JSON
* Why JSON?

---

# Purpose
* Change content for each customer
* A/B testing
* Feature flags
* Less work to make changes

^ Why A/B Test?: Find optimal screen for metrics

---

# Demo

^ Show video

---

# How I made it
* Prototype in sample app
* Use compositional layout and diffable data sources
* Define each section type as JSON w/ a title and subtitle
* Firebase Remote Config

---

# Code Example

^ Explore Shop Project

---

# Challenges/risks
* App Review risks (AKA don’t pull a Fortnite)
* Server unavailable

---

# Other possible approaches
* Defining more in JSON
* [microsoft/AdaptiveCards](https://github.com/microsoft/AdaptiveCards)
* [spotify/HubFramework](https://github.com/spotify/HubFramework)
* Web views

---

# Conclusion
* Easy to strike a balance w/ compositional layout
* Think of important, frequently changing screens
* Worth it for minne
* Try it out in a prototype!
* [yhkaplan/Shop](https://www.github.com/yhkaplan/Shop)

^ Use it on big, complex screens that are important. It’s not worth the architectural burden on less important screens
