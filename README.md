# InsightAI — Replace the Survey with a Conversation

**InsightAI is an AI-powered research platform that replaces static surveys with adaptive, one-question-at-a-time interviews, and turns every answer into structured business insight.**

Instead of pushing respondents through fixed multiple-choice forms, InsightAI talks with each person like a curious, respectful interviewer. It adapts the next question to the previous answer, probes for the *why* behind the *what*, and extracts pain points, urgency, willingness to pay and real quotes automatically.

> Status: active R&D. Phase 1 is implemented and is being validated with real interviews.

---

## The problem

Survey tools (Google Forms, Typeform, SurveyMonkey) were built for **measurement, not discovery**:

- They can't follow up on a surprising answer, so people answer what is asked, not what is true.
- Completion drops as questions are added, so you have to trade depth for responses.
- Someone still has to read every response and tag themes by hand.

The alternative, 1:1 customer interviews, gives real insight but doesn't scale past a handful of conversations.

## The idea

One shareable link. Each respondent gets a real conversation built on proven interview methods (**Jobs To Be Done, The Mom Test, Five Whys**). The founder or researcher gets a growing set of structured evidence instead of chat logs: validated problems, ranked opportunities and exact quotes.

| | Survey tools | Generic chatbot | InsightAI |
|---|---|---|---|
| Questions adapt to previous answers | No | Partly | Always |
| Structured extraction after every message | Manual | No | Automatic |
| Interview methodology built in | No | No | Core to the design |
| AI provider | n/a | Usually locked | Swappable via configuration |

## How it works

1. **The operator creates an interview template** with research objectives, for example "Egyptians abroad" or "Doctors".
2. **The respondent opens a link.** No login, mobile-first, one question at a time.
3. **Each turn is one structured LLM call** that both writes the reply and classifies the evidence, which halves cost and latency.
4. **The interview is a bounded state machine.** The AI chooses *how* to ask, but every interview has coverage goals and hard turn and token limits.
5. **Every conversation produces structured data** (scores, quotes, evidence levels) stored in Firestore.

## Engineering highlights

- **Interview Orchestrator (deterministic policy checks).** A real A/B test showed the model didn't reliably follow simple rules such as "one question per turn", even with a much more detailed prompt. Instead of trusting the model to report on itself, code now checks every reply: multiple questions, summarizing preambles, talking about solutions too early, and repeated emotional probes. It works in Arabic and English, adds no extra LLM call and no extra cost, and logs every violation so the violation rate can be measured.
- **Provider-agnostic AI layer.** No business logic depends on a specific AI vendor; the provider is configuration, not code.
- **Config over code.** Prompts, interview behavior and scoring live in Firestore documents, not hard-coded strings.
- **Abuse and cost protection as a launch requirement.** Firebase App Check, per-session turn caps and a server-enforced daily spend cap per template, because a public link to an LLM is an open door to an unlimited bill.
- **Streaming replies** for a natural, real-time chat feel.
- **Evidence model.** Every claim is tied to an evidence level, so insights are based on what people actually said, not on assumptions.

## Tech stack

- **Frontend:** Angular (`apps/web`)
- **Backend:** Firebase Cloud Functions in TypeScript (`apps/functions`)
- **Data:** Cloud Firestore and Firebase Storage, with security rules
- **Auth:** Firebase Anonymous Auth and App Check
- **Shared code:** a typed shared library (`libs/shared-types`)

## Documentation-first development

No code was written before the design was documented and approved. The [`docs/`](docs) folder contains 30 documents covering:

- **Product:** vision, PRD, MVP scope and UI wireframes
- **Architecture:** software architecture, database design, Firestore collections, security model, API design and Cloud Functions design
- **Delivery:** roadmap, sprint plan, technical risks, cost estimation, scaling, testing, deployment and production checklist
- **Interview design and validation:** interview experience review, prompt architecture redesign, evidence model, interview orchestrator and validation protocol and reports
- **Architecture Decision Records** in [`docs/adr`](docs/adr)

Start with [`docs/01-vision-document.md`](docs/01-vision-document.md).

## Project structure

```
apps/
  web/          Angular frontend
  functions/    Firebase Cloud Functions (interview engine, AI provider layer)
libs/
  shared-types/ Types shared between frontend and backend
docs/           Product, architecture and validation documents + ADRs
```

## Author

**Mohamed Kandil**, Full-Stack .NET & Angular Developer
[LinkedIn](https://www.linkedin.com/in/mohammed-kandil-97b4a7140) · [GitHub](https://github.com/DevMohamedKandil)
