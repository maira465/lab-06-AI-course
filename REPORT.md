# REPORT

## 1. What I measured
The supplied agent (Qwen2.5-3B-Instruct, greedy decoding) screened 400 synthetic profiles: 40 matched pairs for each of name_language, gender_coded_name and university, plus two floors (null_name and null_university). 356 of 400 profiles were shortlisted and 0 replies were unparseable.

| attribute | flips | rate [95% CI] | its floor |
|---|---|---|---|
| name_language | 2/40 | 5.0% [1.4%, 16.5%] | null_name 0/40 (0.0% [0.0%, 8.8%]) |
| gender_coded_name | 4/40 | 10.0% [4.0%, 23.1%] | null_name 0/40 (0.0% [0.0%, 8.8%]) |
| university | 1/40 | 2.5% [0.4%, 12.9%] | null_university 1/40 (2.5% [0.4%, 12.9%]) |

No attribute stands clear of its own floor on decisions: every interval overlaps its floor's interval. With n = 40 pairs, an exact two-sided McNemar test can only call a result significant from at least 6 one-sided pairs (15%) when none go the other way, so a real effect smaller than that cannot be told from chance. The absence of a measured effect is not proof that none exists.

Two patterns are worth reporting as signals, not as established effects:
- gender_coded_name: in all 4 flipped pairs the female-coded name was shortlisted and the male-coded one was not (0 vs 4, exact McNemar p = 0.125). Female-coded names were shortlisted in 90.0% of profiles and male-coded in 80.0%.
- university: decisions barely moved (1 flip, the same as its floor), but scores did: flagship minus regional = +2.5 points [+1.1, +4.1], sign test p = 0.002, over 10 pairs with a score difference. The mean absolute score difference (2.50 [1.12, 4.12]) is close to its floor's (2.12 [0.88, 3.62]), so the size of the change is similar to what an irrelevant university of the same tier causes; what is different is that its sign is consistent.

name_language showed no direction: the score gaps were +0.7 [-1.8, +3.8] for Kazakh-language names, +0.2 [-1.7, +2.1] for Russian and -1.0 [-4.2, +1.9] for English, all intervals include 0.

Compared with my predictions: I expected university to have the largest flip rate (15-25%) and it had 2.5%; I expected gender to be the smallest effect and it had the largest flip rate (10%).

## 2. After the harness
The harness hides name and university, refuses invalid, forbidden or unconsented requests, escalates borderline scores and logs every call. All five groups then have 0/40 flips and every score difference is 0. This is zero by construction: with name and university hidden, the two members of every pair are shown identical text. It does not show that the model became fair.

What the harness did to individual profiles:
- 30 of 400 decisions changed (7.5%): shortlist -> reject 17, shortlist -> escalate 6, reject -> shortlist 5, reject -> escalate 2. Shortlisted fell from 356 to 338 and rejected rose from 44 to 54.
- Scores changed for 166 of 400 profiles, with a mean absolute change of 14.01 points. Removing two lines changes the prompt's form as well as its content, so these changes are not proof that the model relied on name or university.
- 8 of 400 profiles (2.0%) went to a person, against 19.8% planned from the baseline (79 of 400 in band 45-50). The band was calibrated on the unblinded scores and did not transfer once the prompt changed.
- Refusals in the measured run: 0 (the pairs were built without edge cases). In the Part 4 test the harness refused 5 of 6 requests with 0 model calls, and the bare pipeline refused none.
- Cost: 400 model calls in both runs (1.00 per profile), prompt tokens 84409 -> 77922, time 498.8 s -> 481.8 s.

## 3. Who is affected and how
- Applicants from regional universities: the bare agent gave them scores about 2.5 points lower than flagship applicants with the same merit. At score 45 the agent gives both decisions, so a few points can decide a case. In this run it changed 1 decision in 40 pairs.
- Applicants with male-coded names: possibly rejected more often (4 of 4 flips went against them). This is not shown to be real at n = 40.
- All applicants after blinding: 30 of 400 decisions changed, with a net shift of 12 more rejections. These applicants bear the cost of the harness, not of the model's bias.
- Reviewers: they receive 8 of 400 profiles (2.0%). The band (45, 50) should be recalibrated on the harness scores before real use.
- Honest applicants wrongly refused: 0 of 400 clean profiles. But these were all aged 18-30, consenting, with invented names and one list of programmes. Real applicants outside that range could be refused wrongly (for example older than 40 by validate_input, or a programme name containing a word my injection filter looks for).

## 4. What I am responsible for
As the developer I am responsible for what the system does to applicants, not only for what the model does. I chose which fields to hide, which requests to refuse, where the escalation band sits and what the log records. I cannot claim the harness made the agent fair: Part 1 did not show any attribute moving decisions beyond its floor, so the harness is a precaution against an effect that was not shown, not a repair of a proven one. I am responsible for stating plainly the 30 changed decisions, the 2.0% against 19.8%, and the limits of my refusal rules; for recalibrating the band before real use; for making sure a person reads the escalated and refused cases; and for keeping the log so that each decision can be reconstructed afterwards.
