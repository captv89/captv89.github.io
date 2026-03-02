---
title: "Two Halves of the Same Picture — and a Small Experiment with Modern Tools"
date: 2026-03-02T12:00:00+01:00
description: On learning the commercial side of shipping after years at sea, understanding a problem that's already been solved in many places, and building an open-source AI-assisted DA validation tool anyway.
menu:
  sidebar:
    name: OpenDA
    identifier: openda-da-analyzer
    weight: 40
hero: openda_hero.png
mermaid: false
tags: ["Maritime", "Port Agency", "Disbursement Accounts", "AI", "Open Source", "Docling", "LiteLLM", "Human-in-the-Loop", "FastAPI", "Python"]
categories: ["Project"]
---

For most of my time at sea, a port call was something you experienced from the vessel side. The pilot boards, the tugs come alongside, the agent handles everything ashore. What any of that costs, who reconciles it afterwards, and how the numbers get checked — that part of the picture was simply not visible to me.

Joining my current firm on the commercial side changed that. Working across ship owning, operating, and chartering, I got my first real look at Disbursement Accounts — the Proforma DA that goes out before a vessel arrives with estimated costs, and the Final DA that comes back afterwards with the actual invoices and receipts attached. My colleague Gibin, over what became fairly regular lunch conversations, walked me through how the whole process works. It was one of those things that feels obvious once you understand it, but genuinely wasn't until someone explained it properly.

> It was like completing a picture I'd been painting from one side for years.

I should be clear about something before going further. The DA process, as it exists today in most well-run shipping companies, is not the chaos it once was. There are established commercial platforms that handle this properly — streamlined workflows, proper integrations, audit trails. My own firm uses one. The heavy manual reconciliation work that used to define this process has largely been addressed by those tools for operators who have adopted them.

But the maritime industry is wide. Not every port agency, not every small operator, works with modern platforms. The legacy version of this problem — a PDF, a spreadsheet, and someone going line by line — still exists in pockets of the industry. And separately from all of that, I was simply curious: what would this look like if you approached it today, with the tools that are available now, without trying to build another full commercial platform?

## What I Built

[OpenDA](https://github.com/captv89/OpenDA) is an open-source tool — not a platform, not a product — that takes a Proforma DA in JSON format and a Final DA PDF, runs the FDA through an AI extraction pipeline, compares the two, and surfaces the differences for human review. An accountant sees the extracted line items alongside the source PDF with citations highlighted. They correct anything the AI got wrong. An operator does a final sign-off. The approved result fires to a webhook endpoint. That's the whole thing.

The Human-in-the-Loop design was deliberate. AI reading a mix of clean invoices and handwritten chits will get some things wrong or uncertain, and in a financial context that needs to be visible and correctable rather than hidden. The confidence score on each extracted item isn't decoration — it's what determines whether something gets flagged for the accountant to look at.

## The Tools That Were New to Me

More than the domain problem, this project was an excuse to get hands-on with tools I'd read about but not actually used.

### IBM Docling

IBM Research's open-source document understanding library. The reason I chose it over simpler PDF parsers is that it outputs a bounding box for every text element it extracts — exact page coordinates. That made the citation UI possible: clicking a field in the review form scrolls the PDF to the right page and draws a highlight over the source text. It also handles scanned documents and reconstructs tables from images, which matters when your input includes photographed receipts.

### LiteLLM

A unified gateway that abstracts over different LLM providers. Switching between Claude, Gemini, GPT-4o, or a local Ollama model is two environment variables — no code changes. For an open-source project where people might want to use their own provider or run everything locally without an API key, this was the right default to build around.

### react-pdf

The split-screen review UI — PDF on the left, editable form on the right — needed a viewer I could drive programmatically. `react-pdf` wraps PDF.js and handles page rendering as React components, with a canvas layer on top for the bounding box highlights. The coordinate conversion between Docling's output and the rendered pixels was the fiddliest part of the whole project.

## What This Is and Isn't

OpenDA is not trying to compete with commercial DA platforms — those exist, they work, and companies that have adopted them don't need this. It's a focused open-source experiment: a specific slice of the problem, built with current tools, kept simple enough that someone can read the code and understand all of it. If it's useful to a smaller agency or operator who doesn't have a platform, or to someone learning how to build AI-assisted document workflows, that's enough.

A specific thanks to Gibin for the lunch break explanations that made the domain side of this make sense.

The repository is at [github.com/captv89/OpenDA](https://github.com/captv89/OpenDA) — open source under MIT, issues and PRs welcome.
