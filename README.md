# QA Documentation Samples
Sample bug reports and test cases created based on my experience as a software tester — 
All company-specific information has been removed while preserving the original structure,
level of detail, and testing approach.


## Real examples of test cases and bug reports

These are drawn from real work, not written as exercises. Product names, company
names, internal IPs, file paths, and domain-specific field/parameter names have all
been replaced with generic equivalents for confidentiality — but the structure, the
reasoning, and the level of detail are unchanged from how I actually document my work.

## What this shows

The system under test is a web application with direct hardware/PLC integration — an
industrial production-management system. The samples here span a few different kinds
of testing on purpose, rather than just one:

- **Hardware/integration testing** — investigating a Health Check endpoint that continued reporting 
"Status: Connected" after the PLC connection was interrupted, including analysis of the captured 
backend exception trace (Bug Report 1).
- **Frontend defect investigation** — identifying a UI crash and tracing it to the exact source 
location using browser console output (Bug Report 2).
- **API/backend validation** — API/backend validation — verifying server-side validation 
independently of the UI, including testing invalid payloads submitted directly to the API 
and documenting cases where backend validation was missing (Test Case 6, Bug Report 9).
- **Business-logic/scheduling correctness** — boundary testing around overlapping vs. adjacent time slots 
(Test Case 4), not just the obvious full-overlap case.
- **Bug-to-test-case traceability** — documenting multiple defects uncovered during a single test 
case as individual linked bug reports rather than combining them into one generic issue 
(Test Case 1 → Bug Reports 4–7).

## Contents

`sample-bug-reports-test-cases.md` — 9 bug reports, 6 test cases, each following
a consistent format: Priority, Severity, Environment, Preconditions, Steps, Expected
vs. Actual results, and links between test cases and the bugs they surfaced.
