# Assignment Announcement — System Testing (Panda-Lite API)



## Presentation
* **Section B, C: TBA**
* **Section A, D: TBA**
    * Each team will do a short in-class presentation of the key defects they found.
    * For each defect, take a screenshot of the evidence (request/response, etc.) and add it to the slides — **one slide per defect is enough**, though you may use multiple slides if a scenario needs extra steps to explain.

## What you'll do

You will perform **system testing** on the **Panda-Lite API** — a lightweight food-delivery system that handles users, restaurants, menu items, orders, and ratings, secured with **JWT-based authentication** and backed by **PostgreSQL**.

* **Before you start:** watch the class videos and tutorials first — they cover the concepts and workflow you'll need for this assignment.
* **Specifications & how to run:** Everything you need is in **`documentation.md`** — endpoints, expected behaviors, and setup instructions.
* **Do not** copy API specs into your report. Just **reference `documentation.md`**.

## Important context

The API may have bugs — some intentional, some not. **Your job is to find them, reproduce them, and report them.** That's it. You're not here to fix the code.

Use whatever tools work for you — Postman, REST-assured, custom scripts, anything.

> Quality over quantity. A handful of well-documented, reproducible defects is worth far more than a long list of shallow ones.

> [!CAUTION]
> ## Collaboration & Plagiarism Policy
>
> **This section is strictly enforced. Please read it.**
>
> There's a clear line between what's allowed and what isn't:
>
> * You can talk to other teams about general approaches — tool setup, how to structure a test plan, how JWT works.
> * **Do NOT share** test cases, test data, or specific test ideas with anyone outside your team — not verbally, not in writing, not in any form.
> * **Do not use AI tools**, the internet, or peers to generate your test cases or defect ideas. Your work must come from your own hands-on analysis of the system.
> * **Do not copy or paraphrase** anything from external sources. Rewriting someone else's test case in your own words is still plagiarism.
> * If your team is caught sharing or receiving specific testing content, **everyone involved will be penalized** — regardless of who started it.
>
> If you're unsure whether something is okay, it probably isn't.

## Deliverables

Grading will be split between **report quality** and **testing coverage/effectiveness** — both matter.

Submit one **Markdown report** and a Postman collection export. Your report must have these sections:

1. **Test Plan**
    * For each endpoint or behavior, document your input choices and what you expect back (output + HTTP status code). Use the class slides as a guide.
    * Test specification for each function/endpoint.

2. **Test Cases**
    * Use a table or list format with: **ID, Short Title, Pre-conditions, Steps, Expected Status Code, Actual Status Code, Verdict**.
    * The **ID** for each test case "can" be the **last 6 digits** of the `requestId` from that request/response's `_lab` field.
    * Cover the full range: happy paths, bad inputs, auth failures, permission checks, ownership rules, order lifecycle transitions, cancellation and rating restrictions, filter behavior, and edge cases.

3. **Defect Reports**
    * For each defect: ID, Title, Severity, **Steps to Reproduce**, Expected, Actual, Test Case No.
    * The defect **ID** is also the **last 6 digits** of the `requestId` from the `_lab` field of the response where the defect was observed.
    * Group related defects together and flag cascading or duplicate issues where relevant.

4. **Individual Reflections** *(1 page total for the whole team)*
    * A short paragraph from each team member on what they personally contributed to the testing effort.

### Evidence
Export your Postman collection (with all requests/responses) as a JSON file and upload it alongside your report — this is your evidence for test cases and defects. No separate screenshots or evidence folder needed for the report itself.

## Bonus

Build something genuinely useful — a reusable test harness, a scenario runner that covers auth + order lifecycle + permissions, a fuzzer, anything that goes beyond a basic fetch-and-check script — and you may earn bonus credit. Include the repo link in your report.

## Submission

Name your archive exactly as specified in ELMS:
**`<StudentID1>_..._<StudentIDn>_SystemTesting.zip`**

Inside:
* `<StudentID1>_..._<StudentIDn>_SystemTestingReport.md`
* Postman collection export (`<StudentID1>_..._<StudentIDn>_SystemTesting.json`)

## A Few Last Things

* Every defect should be reproducible and backed by evidence.
* When in doubt about anything technical, check **`documentation.md`** first.
