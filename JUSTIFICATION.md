# JUSTIFICATION

## 1. What the system may do
The system may screen scholarship applications for one purpose only: `scholarship_screening`. It may process adults who have consented to automated processing, using merit and need fields (programme, age, GPA, entrance exam score, household income bracket, achievements). Name and university are not shown to the model. The model's output is advisory: shortlist, reject and its score are recorded, and cases the harness flags go to a person. In the vocabulary of Law No. 230-VIII Art. 17(2), the system is designed between low autonomy (a person makes the final choice) and medium autonomy (a person can correct or cancel the decision).

## 2. What it must refuse
It refuses before any model call, and logs the refusal with a reason code:
- `purpose_not_allowed`: any purpose other than scholarship screening (the test request "rank applicants by ethnicity");
- `minor`: an applicant under 18;
- `no_consent`: no consent to automated processing;
- `invalid_input`: impossible values (GPA 9.9, exam score -5);
- `suspected_prompt_injection`: a free-text field that looks like an instruction to the model (my own rule).

In the Part 4 test the bare pipeline screened all 6 requests and refused none; the harness refused 5 of 6 with 0 model calls, and the ordinary application got 1 model call and a normal decision.

Limits of the refusal rules, which I accept and state:
- The injection rule is a keyword and character filter. It can wrongly refuse a legitimate programme or university name that contains a word such as "score" or "prompt", and it can miss an instruction written in another language or with other words.
- `validate_input` refuses any applicant older than 40, including an honest mature student.
- The minor rule refuses a real 17-year-old applicant. That is intended, but it still excludes that person from this channel.
- Of the 400 clean measured profiles, my three refusal checks would refuse 0. This is weak evidence: they were all aged 18-30, all consented, and their names and programmes come from one invented list.

## 3. What record it keeps
`decisions.jsonl` has one JSON line per call: 406 records (400 measured profiles and 6 test requests). By outcome: shortlist 339, reject 54, escalate 8, refuse 5. Each record holds a record id, the application id, the SHA-256 of the exact text the model saw, the fields shown and withheld, the model and prompt version, the harness settings, the raw reply, the decision and score, the controls that fired, the outcome and reason code, whether it went to a person, and a timestamp.

The log holds field names, not field values, so it does not become a second copy of the applicants' data. A reviewer can see why a decision was made (reason code, controls fired, the model's own reply) and can check that a given input produced it (the hash), without re-running the model. A refusal record says `[no model call]`, so it is clear that the model was never asked.

Trade-offs of the log: a reviewer cannot read the applicant's values from the log alone, so the original applications must be stored separately under access control. The raw reply may quote values the model was shown, so the log itself still needs protection.

## 4. Controls, principles and law
See `control_register.json`. Each control that is switched on (validate_input, blind, refuse, log, escalate) is tied to a named principle of the UNESCO Recommendation on the Ethics of Artificial Intelligence (2021) with its paragraph numbers, and to a named provision of Law of the Republic of Kazakhstan No. 230-VIII "On Artificial Intelligence" of 17 November 2025 (Art. 6, 8, 9, 11, 15, 16, 17) and, for consent, Law No. 94-V "On Personal Data and Their Protection" of 21 May 2013 (Art. 8).

Two limits of these citations. First, the UNESCO Recommendation is adopted by Member States "on a voluntary basis" (paragraph 2 of its operative text), so I use it as an ethical source, not as a binding rule. Second, this system is a laboratory prototype that handles invented applicants; I do not claim that Law No. 230-VIII binds it, and I use the law as a benchmark for what a real deployment of this kind of system would have to meet. I read the English text of Law No. 230-VIII (an unofficial translation) and the text of the UNESCO Recommendation; for Law No. 94-V I read only Art. 8 as found in a published text, so a lawyer should check it against the current official version before real use.

## 5. Trade-offs I accepted
- **Blinding name and university.** It removes the model's chance to use them, at no extra compute (400 model calls in both runs, prompt tokens 84409 -> 77922). But it changed 30 of 400 decisions (7.5%) and tilted them towards rejection (shortlisted 356 -> 338, rejected 44 -> 54), and it discards any legitimate signal in the university. Removing lines changes the prompt's form as well as its content, so I cannot say the model relied on those fields.
- **The escalation band (45, 50).** I chose it from the baseline scores: 45 is the only score at which the model gave both decisions, and 50 is the next step above. It held 79 of 400 baseline profiles (19.8%). After blinding only 8 of 400 (2.0%) were sent to a person, because the scores moved. A wider band means more reviewer work; a narrower one means more decisions nobody re-checks. The band must be recalibrated on the harness scores before real use.
- **Counterfactual check and order probe.** I did not run either extension (the check doubles the model calls), so the agent's sensitivity to the layout of the prompt is not measured.
- **What Part 1 could and could not show.** No attribute stood clear of its own floor on decisions, and n = 40 can only detect 6 or more one-sided pairs (15%). The harness is therefore a precaution against an effect that was not shown, not a repair of a proven one.

## 6. Which control was hardest to tie to a provision
My guess before searching: `blind` and `escalate`, because I expected the laws to state broad principles and no norm about hiding a field or a score band.

After reading the text of Law No. 230-VIII, I was partly wrong. `blind` ties to Art. 6 (no discrimination based on language, gender, origin or any other circumstances) and `escalate` ties to Art. 17(2), which describes systems where a person makes or can correct the final decision. The hardest controls to tie to a specific provision were the purpose rule (`purpose_not_allowed`) and the injection rule: the AI law has no norm that names purpose limitation or prompt injection, and the closest provisions (Art. 11(1) on safety, and Art. 15(2)(2) on protection from unauthorised access and failures) are general. For `log`, Art. 8(1), 15(2)(3) and 16(1)(5) cover control, documentation and a user's right to request the data behind a decision, but none of them says what a decision log must contain; I chose that content myself.

## 7. AI-use declaration
I used Claude (Anthropic) as an assistant in this lab. It helped me understand what each notebook cell asked for; it drafted wording for my predictions and observations (PREDICTION_0, 1, 3, 4 and OBSERVATION_1, 3, 4), the first versions of my refusal rule and my escalation rule, and drafts of REPORT.md and this file; it suggested the escalation band, and it suggested which legal instruments to look up and found the text of Law No. 230-VIII and of the UNESCO Recommendation. I ran all the code in the notebook and all the numbers come from its output. [EDIT THIS PART TO BE TRUE: say what you changed yourself, and which articles and paragraphs you opened and read yourself.]
