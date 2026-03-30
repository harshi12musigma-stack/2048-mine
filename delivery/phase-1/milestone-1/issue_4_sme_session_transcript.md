# Issue #4 — SME Walkthrough: Full Session Transcript
**Project:** alpha-test | **Client:** Mars India
**Phase:** 1 — Requirement Gathering & Business Understanding
**Date:** 31 March 2026
**Document type:** SESSION TRANSCRIPT — source material for Issue #4 deliverable

---

## Attendees

| Person | Role | Company | Session |
|---|---|---|---|
| Harshita Gupta | Project Lead, Analytics | Mu Sigma | W-01, W-02, W-03 |
| Vishwavani | Data Scientist | Mu Sigma | W-02 |
| Savikar | Data Engineer | Mu Sigma | W-03 |
| **Priya Sharma** | Head of CRM & Retention, India | Mars Petcare | W-01 |
| **Karthik Nair** | Analytics & Data Lead | Mars India | W-02 |
| **Rahul Mehta** | IT Platform & Infrastructure Lead | Mars India | W-03 |

---

---

# W-01 — Business Process Walkthrough
**Session with:** Priya Sharma, Head of CRM & Retention, Mars Petcare India
**Mu Sigma:** Harshita Gupta
**Duration:** ~65 minutes
**Format:** Video call (Teams)

---

**[00:00 — Intro]**

**Harshita:** Hi Priya, thanks for making time — I know mornings are hectic. Can you hear me okay?

**Priya:** Yes, you're clear! Sorry I'm a minute late, we just had a quick stand-up that ran over. Okay I'm here, let's go.

**Harshita:** No worries at all. So just to set context — this is not an audit or evaluation of anything you're doing today. We genuinely want to understand how the retention process works right now, end-to-end, so that whatever we build fits into how your team actually operates, not some theoretical workflow. So I'll mostly ask you to walk me through things as if I've never seen your world before.

**Priya:** That's helpful to know, honestly. Some vendors come in and immediately want to show you a demo of what they're going to build. So yes — ask away.

---

**[04:30 — Section A: Current At-Risk Identification]**

**Harshita:** Okay, let's start at the beginning. Walk me through — on a typical Monday morning, how does your team know which customers are at risk?

**Priya:** Right. So currently... and I'll be honest, it's not very sophisticated. We have a report that Karthik's team runs — it's basically a SQL query that pulls anyone who hasn't purchased in the last 45 days, and tags them as "follow-up required." That runs every Monday morning, we get it as an Excel file.

**Harshita:** Forty-five days — is that a hard rule? Like, exactly 45?

**Priya:** It started as 45 but honestly it's shifted around over time. At one point someone changed it to 60, then it went back to 45 for Royal Canin because we felt we were catching people too late at 60. So right now RC is 45 days and Pedigree Club is... I think still 60? I'd have to check with Karthik.

**Harshita:** That's really useful, actually. So the two segments already have different thresholds operating.

**Priya:** Yes, which is a problem because the Excel sheet doesn't distinguish them cleanly, so Monday mornings my team is manually filtering it. It takes about two hours before the list is actually usable.

**Harshita:** Okay. And who runs this report — does it run automatically or does someone trigger it?

**Priya:** Karthik's team schedules it. But it's a cron job that emails a file, and sometimes the file is late or doesn't come through and we have to ping them. It's not — I mean, it works, but it's fragile. Two weeks ago it didn't run because of some server maintenance and we didn't realise until Wednesday.

**Harshita:** *(making a note)* Lost two days on the retention window. And when you say it takes two hours to make usable — what's happening in those two hours?

**Priya:** We're filtering by segment, removing customers who we've already contacted in the last two weeks so we're not double-pinging them, removing customers who have a pending delivery — because they haven't purchased but they have something coming, so they're fine — and then roughly prioritising by spend. It's all manual in Excel.

**Harshita:** Who does that? Is it you, or is it someone on your team?

**Priya:** One of my coordinators, Deepa. She's very good at it by now, honestly she could do it in her sleep. But she's the only one who knows the full filter logic. If she's on leave, it doesn't happen.

**Harshita:** That's a critical single point of knowledge. Does that logic live anywhere — documented, or just in Deepa's head?

**Priya:** *(laughs)* Deepa's head. And maybe a WhatsApp message from six months ago when we first set it up.

**Harshita:** Okay. When do you consider a customer officially "churned"? Is it when they hit 45 days, or is there a harder definition?

**Priya:** That's actually something I want your help with, because we've never formally defined it. Internally we say "lapsed" at 45 days and "churned" after 90, but those terms aren't consistent across teams. Marketing might say someone is churned after 60 days, Finance has some other definition for their churn KPIs — I think they use 120 days. It's a mess.

**Harshita:** Noted — that's something we'll need to pin down before we build anything. The model's entire training depends on having one stable label definition.

**Priya:** Yes, and that's something I've been trying to get aligned on for two years. Maybe the fact that Mu Sigma is asking will force the conversation. *(laughs)*

---

**[22:00 — Section B: Retention Actions]**

**Harshita:** Let's talk about what happens once you have the list. A customer is flagged — what happens next?

**Priya:** So we have three tiers of action. For Royal Canin Direct — the vet-referred customers — the sequence is: first we send a WhatsApp message at day 45. Personalised-ish, we have templates, but the vet's name is in it, which helps. Then if there's no response in five days, we send an email. Then if still nothing at day 55, we escalate to an outbound call from our retention team. We have seven people on the retention desk.

**Harshita:** How many customers can those seven handle per week?

**Priya:** On calls? Realistically about 4,500 to 5,000 per week across the team. Some weeks it's lower if there's training or someone's sick.

**Harshita:** And what's the volume of at-risk customers you're currently flagging weekly for RC Direct?

**Priya:** It varies. Peak is maybe 3,000 a week after a long weekend or festive season. Normal weeks, around 1,200 to 1,500.

**Harshita:** So capacity isn't a constraint right now for RC Direct.

**Priya:** For RC Direct, no. For Pedigree Club, yes. Pedigree Club has maybe eight times the customer base but we can't call all of them — so for Pedigree we mostly do email and WhatsApp only, no calls unless it's a high-value account.

**Harshita:** What defines "high-value" for Pedigree?

**Priya:** Honestly, Gold tier or above, or anyone who's spent more than twenty thousand rupees in the last year. That's a rough cut my manager set, it's not a model or anything.

**Harshita:** Got it. And the WhatsApp and email — are those going through Salesforce Marketing Cloud?

**Priya:** Email yes, SFMC. WhatsApp is through a third party — we use Kaleyra through our WABA number. They're not integrated, which is another pain point. So my team has to manage two campaign tools and manually reconcile who responded to what.

**Harshita:** Do you currently track whether the outreach worked?

**Priya:** We track re-purchase. If someone in the at-risk list buys within 30 days of us contacting them, we count that as a retention save. But we don't have a holdout — so we don't actually know if they would have come back anyway. Our "retention rate" right now is probably inflated.

**Harshita:** That's very honest. What does your current retention rate look like on paper?

**Priya:** On paper, about 22% for RC Direct and maybe 14% for Pedigree. But without a control group — *(shrugs)* — I don't fully trust that number.

**Harshita:** For context, the data we've seen shows roughly 31% of RC Direct has churned in the sample. Does that align with what you're seeing?

**Priya:** Yes, that's in the right ballpark. Maybe slightly higher some months. It's the segment that worries me most because the LTV gap is so large. Losing one RC Direct customer is not the same as losing one Pedigree customer.

---

**[41:00 — Section C: Pain Points]**

**Harshita:** What's the biggest frustration with how things work today?

**Priya:** *(immediately)* We find out too late. By the time someone hits 45 days, they've mentally already decided. I want to know at 20 or 25 days — when there's still something we can do. The data is there, I'm sure — they've stopped opening emails, stopped clicking, their WhatsApp messages are delivering but not being read — but we have no way to act on that right now.

**Harshita:** That matches what we see in the data exactly. All the churned customers have near-zero email open rates and zero WhatsApp engagement in the 90-day window before they churned — but that signal was building for weeks before the 90-day mark.

**Priya:** Exactly! And I get reports on email open rates, but they're aggregate reports — overall campaign open rates. I can't look at individual customer engagement trending downward. It's just not possible in the current setup.

**Harshita:** Any customers you know are at risk but the current process misses?

**Priya:** Yes — what I call the "ghost purchasers." These are customers who buy occasionally, never regularly, so they never trigger the 45-day rule because their baseline is already low. They haven't bought in 70 days but for them that's normal. And then they disappear. We don't catch them because we're comparing to a fixed threshold, not to their individual purchase cadence.

**Harshita:** That's a really important edge case. The model needs to be trained on relative frequency, not just absolute days.

**Priya:** Yes. And then the opposite — someone who usually buys every two weeks and suddenly goes 20 days without buying. Current system won't flag them for another 25 days, but I'd want to know about them now.

**Harshita:** False positives — are you over-contacting anyone?

**Priya:** Yes. We contact a lot of customers who've just changed their purchase channel. They moved from our portal to Amazon or a vet clinic and they're still buying Royal Canin, just not from us directly. We flag them as at-risk, call them, and they're confused. *(laughs)* "I just bought it yesterday, why are you calling?" That's embarrassing and it erodes trust.

**Harshita:** Can you detect channel switches in your current data?

**Priya:** Not systematically. Sometimes a vet will call us and tell us they've referred X customers who are now buying at their clinic. But we have no automated way to know. It's a data problem.

---

**[54:00 — Section D: Output Requirements]**

**Harshita:** If we gave you a weekly scored list — customer ID, risk score, a risk tier, maybe a recommended action — what would you actually do with it?

**Priya:** Okay so first — it needs to be in Salesforce. Or at minimum it needs to be something I can import into Salesforce without IT involvement. Because if it's a Snowflake table and I need to raise a ticket every Monday to get it exported, that's not going to work.

**Harshita:** Noted. So the output landing zone is Salesforce, not just Snowflake.

**Priya:** For the business side, yes. For Karthik's team, Snowflake is fine. But for the CRM team — Salesforce. We can segment directly in SFMC from a Salesforce object.

**Harshita:** What's your minimum lead time — when do you need the scores by to run that Monday campaign?

**Priya:** The campaign brief has to go to SFMC by Tuesday 6 PM for a Wednesday send. So scores need to be in Salesforce by Monday end of day at absolute latest. Sunday night scoring would be ideal.

**Harshita:** That's very actionable threshold. And risk tiers — if we give you High, Medium, Low, is that enough? Or do you need more granularity?

**Priya:** Three tiers is fine. Actually, I want a fourth: "Do not contact." For Gold and Platinum customers who are behaving normally — I don't want my team calling them unnecessarily.

**Harshita:** Interesting — so a suppression tier.

**Priya:** Yes. Because currently my team doesn't have that. They'll sometimes call a perfectly happy Platinum customer because they showed up on a list and it creates friction.

**Harshita:** Final question — who else on your team would use this output?

**Priya:** Deepa, who currently processes the list. My manager, Rohan — he reviews the weekly retention summary. And the Kaleyra WhatsApp campaign manager, though that's more of a downstream consumer. Rohan mainly needs a dashboard view, not the raw scores — he wants: how many at-risk this week, how many we acted on, what was the recovery rate.

**Harshita:** Okay — so we're looking at two output types. An operational scored list for Deepa and the campaign team, and an executive summary dashboard for Rohan.

**Priya:** Exactly. That's exactly it.

**Harshita:** This has been incredibly useful, Priya. Really. We have a much clearer picture of the current state. I'll send through a summary of what we've captured and flag the open items.

**Priya:** Please do. And — one last thing — Deepa is actually excited about this. She's been manually doing this for two years. She'd love to just walk into Monday morning with a clean list. So I'd love to involve her in UAT when that time comes.

**Harshita:** Absolutely, we'll plan for that. Thank you so much.

---

---

# W-02 — Data & Analytics Walkthrough
**Session with:** Karthik Nair, Analytics & Data Lead, Mars India
**Mu Sigma:** Harshita Gupta + Vishwavani
**Duration:** ~58 minutes
**Format:** Video call (Teams), screen share for data architecture diagram

---

**[00:00 — Intro]**

**Harshita:** Karthik, thanks for joining. I have Vishwavani with me — she'll be the DS lead on the modeling side, so some of the feature engineering questions will come from her.

**Karthik:** Perfect. I've got ten minutes before the hour once we finish — so I'll try to be focused. I've reviewed what Priya shared with you in context, so I know roughly where you are.

**Vishwavani:** Great. I'll probably jump in when we get to the data specifics, if that's alright.

**Karthik:** Please.

---

**[02:30 — Where does data live?]**

**Harshita:** Starting with the basics — where does the customer data actually live right now?

**Karthik:** So we have three main systems. Salesforce CRM is the customer master — registration, segment, region, all profile data. Transactions are in an Oracle database — it's the e-commerce backend that powers the Mars Direct portal. And the loyalty data — points, tiers, redemptions — lives in Capillary's platform, which is a third party. We get that as a daily SFTP dump.

**Harshita:** And is any of this in Snowflake already?

**Karthik:** Salesforce and Oracle are in Snowflake. We use Fivetran for both. Capillary is not — it's still SFTP files landing on an S3 bucket and our team has a Python script that loads them into a schema in Snowflake. It works but it's not robust. Any format change on Capillary's end breaks it, and they've changed the file format twice in the last year.

**Vishwavani:** Good to know. Is that Snowflake schema production-quality or more of a landing zone?

**Karthik:** Landing zone. There's no data quality checks. It's basically raw files in a table. We haven't had bandwidth to build proper transformation logic on top of it.

**Vishwavani:** And the aggregations we see in the extracted sample — the 90-day purchase counts, 12-month spend — where are those computed?

**Karthik:** Those are computed in a dbt project I built. We have a `marts` schema in Snowflake where we maintain customer-level fact tables. The 90-day and 12-month windows are recomputed daily by a dbt job. But — and this is important — the job runs on a full refresh model, not incremental. So it rescans the full transaction history every day. On a medium warehouse it takes about 90 minutes.

**Vishwavani:** That's a significant compute cost if we're adding feature columns on top of that.

**Karthik:** Yes. That's my concern too. If you add feature engineering on top of a 90-minute full-refresh dbt job, we're looking at overnight runs that bleed into morning. We need to think about that carefully.

**Vishwavani:** Agreed. We'd probably want to move to incremental models for the churn feature mart specifically.

**Karthik:** That's the correct answer, and I've wanted to do it for months. This might be the forcing function.

---

**[18:00 — Modeling history]**

**Harshita:** Has any ML modeling been done on this data before — internally or with another vendor?

**Karthik:** *(pauses)* Yes, briefly. About a year ago — early 2025 — we had an internal data scientist, Manish, who built an XGBoost model. It looked good on paper. Ninety percent accuracy on the test set.

**Vishwavani:** What happened?

**Karthik:** The test set happened. *(laughs bitterly)* He trained it on data where churn had already occurred and then used features that were only available after the churn event. Classic leakage. The model was basically predicting something it already knew. When we tried to score live customers, the precision was terrible — maybe 15% at the top decile.

**Vishwavani:** Subscription cancellation used as a feature?

**Karthik:** Exactly that. And also last purchase date as a raw feature rather than as a relative window. The model was essentially learning "days since last purchase is large → churned," which you can do with a rule. Manish left the company six months ago, so the model was never deployed. We still have the notebooks though if you want them.

**Vishwavani:** Please, yes — they're useful even as a "what not to do" reference and to understand what signals he was testing.

**Karthik:** I'll share them after this call.

**Harshita:** And the current state is the rule-based system I heard about from Priya? The 45-day threshold?

**Karthik:** Correct. That's what's in production. It's a Redash query that emails a CSV every Monday. It's not versioned, it's not tested, it has no monitoring. But it runs.

---

**[31:00 — Customer population & data quality]**

**Harshita:** What does the actual customer population look like? Our sample was 80 rows — how many customers are we really talking about?

**Karthik:** Royal Canin Direct — active accounts in India — around 285,000. Pedigree Club loyalty members — about 1.1 million. But of those, about 40% have been inactive for more than 180 days and are probably already lost for all practical purposes. So the scoreable active population is closer to 170,000 RC Direct and 650,000 Pedigree Club.

**Vishwavani:** That's a manageable size for weekly batch scoring. No volume issues there.

**Karthik:** The volume is fine. The data age is the issue. Historical transaction data before 2022 is on a legacy system that wasn't fully migrated. There are some gaps — specific months in late 2021 where records are missing. If you're building training labels that go back more than three years you'll hit those gaps.

**Vishwavani:** So practical training window is 2022 onwards.

**Karthik:** Safe to say 2022 onwards, yes. Three-plus years of clean data — that should be enough.

**Vishwavani:** One thing I noticed in the sample — all churned customers have zero WhatsApp engagement. Is that real in production?

**Karthik:** It's real in a noisy way. We have read receipts sent back from Kaleyra but they're not 100% reliable. There are some customers marked as "not engaged" who probably received and read the message but the webhook didn't fire. I'd treat WhatsApp engagement as directionally correct but not precise.

**Vishwavani:** Understood. Use it as a feature but don't over-rely on it in the model.

**Karthik:** Exactly. Email open rate from SFMC is much more reliable — those are pixel-tracked.

**Harshita:** Identity fragmentation — Priya mentioned customers who switch from the portal to buying on Amazon or at a vet clinic disappear from our data. How bad is that problem at the ID level?

**Karthik:** We have an ID bridge table — `CUST_MASTER.ID_BRIDGE` — that tries to link the same person across Salesforce CRM, the portal order DB, and the Capillary loyalty platform. Coverage is around 68% for RC Direct, probably 55% for Pedigree because the loyalty enrolment is more casual. So for about a third of our customers we may be working with a partial view.

**Vishwavani:** That's actually something we should address in the feature engineering — flagging records with incomplete identity coverage so the model score comes with a confidence indicator.

**Karthik:** Good idea. I've never had bandwidth to do that but it would be very useful.

---

**[50:00 — Data refresh cadence]**

**Harshita:** How fresh is the data at any given time? If we're scoring on Sunday night, what's the last data touch point?

**Karthik:** Fivetran runs for Salesforce and Oracle every night at 2 AM — so by Sunday 3 AM, Friday's transaction data is in Snowflake. Saturday and Sunday transactions won't be there until Monday 3 AM. So Sunday night scores are always about 1–2 days stale on transactions.

**Vishwavani:** That's acceptable for weekly scoring — the 90-day window means 2 days of lag is negligible.

**Karthik:** Agreed. But if you ever go to daily scoring, you'd need to push Fivetran to real-time or near-real-time, which is a different conversation with IT.

**Harshita:** Noted. That's a Phase 2 problem. One last thing — are there data sources in production that aren't in our current sample that we should know about?

**Karthik:** Two things. One — we have a customer service ticket system, Freshdesk. There's complaint history there. I didn't include it in the initial data pull because the integration isn't clean, but if a customer has opened three complaints in a year and then goes silent, that's probably a churn signal. I can add that.

**Harshita:** We actually have `num_complaints_12m` in the sample already — is that from Freshdesk?

**Karthik:** Yes, that was my quick export. The column's there but the integration for production isn't built. The current sample was manual.

**Vishwavani:** Okay, so it exists but needs a production ingestion pipeline built.

**Karthik:** Correct. Second thing — we have some Nielsen panel data. Aggregate category-level switching signals, not individual. I don't think it's useful for individual churn prediction but if you want to use it as a macro signal it's available.

**Vishwavani:** Probably not for Phase 1. Useful later for cohort-level analysis, not individual scoring.

**Karthik:** That's my feeling too.

**Harshita:** Karthik, this was excellent. Exactly what we needed. I'll coordinate with you on the notebook handover and we'll schedule a follow-up once we've profiled the Snowflake environment.

---

---

# W-03 — IT / Platform Walkthrough
**Session with:** Rahul Mehta, IT Platform & Infrastructure Lead, Mars India
**Mu Sigma:** Harshita Gupta + Savikar
**Duration:** ~48 minutes
**Format:** Video call (Teams)

---

**[00:00 — Intro]**

**Harshita:** Rahul, thanks. Savikar is with me — he's the data engineer who'll be building the pipeline, so the infrastructure questions will mainly come from him.

**Rahul:** Sure. I'll preface this by saying — whatever access we provide to Mu Sigma goes through our standard vendor onboarding. DPA first, then access. I'm happy to describe the architecture but nothing gets provisioned until the legal paperwork is done.

**Savikar:** Completely understood. We're just designing right now — not requesting access yet.

**Rahul:** Perfect, then let's go.

---

**[03:00 — Snowflake]**

**Savikar:** Starting with Snowflake — what edition are you on and what's the region?

**Rahul:** Enterprise edition, AWS ap-south-1, Mumbai. We have a production account and a non-prod account. The non-prod account has dev and staging schemas within it.

**Savikar:** Are dev and staging isolated at the database level or just schema level?

**Rahul:** Schema level within the same non-prod account. Dev schema is `MARS_DEV`, staging is `MARS_STG`. Production data lives in a separate account entirely — `MARS_PROD`. Different credentials, different access controls.

**Savikar:** And the staging data — is it masked? If I'm testing feature engineering on staging, will PII fields be pseudonymised?

**Rahul:** Staging has tokenised customer IDs and masked email/phone — same as what's in your sample data. Names are replaced with generics. Address fields are region-level only, not street level. For all practical ML feature engineering you shouldn't need any of the masked fields.

**Savikar:** Good. And for Mu Sigma access — are we looking at a dedicated service account, or individual user accounts?

**Rahul:** Service account for pipelines, individual named accounts for analysts. The pipeline service account will have read access on source schemas and write access on a dedicated `MU_SIGMA_DEV` schema that we'll provision in MARS_DEV.

**Savikar:** And warehouse sizing — what compute is available for our pipeline runs?

**Rahul:** We'd start you on an X-Small warehouse with manual scaling. If you need more for the dbt feature mart runs you raise a request and we review. We don't auto-provision compute to vendors — everything goes through a change request.

**Savikar:** Understood. X-Small will be fine for development. We'll flag when we need to size up for production.

---

**[15:00 — Airflow]**

**Savikar:** What's the Airflow setup?

**Rahul:** MWAA — Amazon Managed Workflows for Apache Airflow. Version 2.8.1. It's running in the same AWS account as our Snowflake Fivetran setup.

**Savikar:** Who has DAG deployment access currently?

**Rahul:** The analytics platform team — Karthik's side — has deployment access to the dev environment. Production DAG deployments require a change ticket reviewed by me and approved by the IT architecture board. Currently that review cycle takes 5 to 7 business days.

**Savikar:** That's important for our Phase 1 timeline. We need to account for that window before any production deployment.

**Rahul:** Yes. And be aware — the MWAA environment has a requirements.txt that's centrally managed. If you need Python packages that aren't already installed, you submit a package request. We evaluate it for security — no PyPI packages with known CVEs, no packages that require native system libraries that MWAA's managed environment doesn't support.

**Savikar:** What Python version is the MWAA running?

**Rahul:** Python 3.11. Airflow 2.8 on MWAA uses 3.11.

**Savikar:** And standard ML packages — scikit-learn, LightGBM, pandas, numpy — are those already in the requirements?

**Rahul:** Pandas and numpy, yes. Scikit-learn, yes. LightGBM — not currently installed. You'd need to submit a package request. Turnaround is usually 3 to 5 days once we validate the package.

**Savikar:** Noted. We'll submit that early.

---

**[26:00 — Data Sharing & DPA]**

**Harshita:** Let's talk about the DPA specifically. Where is it in the process?

**Rahul:** Our legal team received Mu Sigma's standard vendor agreement last week. There are two clauses under review — the data residency clause, because we require all customer data to remain in India-region infrastructure, and the sub-processor clause, which covers whether Mu Sigma can use any cloud tools on your side to process Mars data. If Mu Sigma is doing any model training on your own AWS or GCP environment, that needs separate approval.

**Harshita:** Our intention is to train and run everything inside Mars's Snowflake environment — not move data out.

**Rahul:** That simplifies things considerably. If it's all within our Snowflake account, the sub-processor clause becomes much simpler. Legal should be able to turn that around faster. I'd estimate DPA completion in three to four weeks from now if there are no further blockers.

**Harshita:** Three to four weeks. So data access realistically in late April?

**Rahul:** That's optimistic but possible. I'd plan for first week of May as the safe assumption.

**Savikar:** That means our DEV environment setup can start in May, and we should use April for design and specification work — not waiting on access.

**Rahul:** Correct. You can design DAG structure, feature specifications, schema design — none of that requires live access. Just don't start writing pipeline code against live data until the DPA is signed.

---

**[37:00 — Dev/Staging/Prod environments & deployment process]**

**Savikar:** Walk me through the change management process for production deployment. What does it look like step by step?

**Rahul:** Development in MARS_DEV — no process, you commit your DAG to a dev branch. Testing in MARS_STG — you raise a pull request, our platform team reviews and merges, automated tests need to pass. Then for production — you raise a formal change request in ServiceNow, it goes to the IT architecture board, they review the DAG logic, the Snowflake queries, and any new packages. Then there's a one-week UAT window with Mars's analytics team reviewing outputs. Then production deployment on an approved maintenance window — which is typically Sunday night.

**Savikar:** So start-to-finish for a production deployment could be three to four weeks from code complete?

**Rahul:** In the worst case, yes. In practice, if the change request is well-documented and the analytics team signs off quickly on UAT, it can be closer to two weeks.

**Savikar:** That timeline affects our Phase 1 delivery significantly. If the June 30 deadline is hard, we need code complete by early June to allow two to three weeks for the production deployment process.

**Rahul:** That's correct. I'd strongly recommend you raise the change request the moment the model and pipeline are stable in staging — don't wait for perfect. The review process can run in parallel with your final testing.

**Harshita:** Noted. That's a real constraint we'll build into the project plan.

---

**[44:00 — Any other constraints]**

**Harshita:** Any other constraints we should know about that aren't in scope of the standard questions?

**Rahul:** Two things. First — Royal Canin Direct customers have health-related purchase data — prescription diet, vet referral records. Our InfoSec team has flagged those fields as requiring separate data classification review before they can be used as ML features. We're expecting a ruling from Legal in the next two to three weeks. Until then, the vet referral flag and any prescription-related fields should be treated as off-limits.

**Savikar:** So `vet_referred` in our current feature set is at risk of being excluded?

**Rahul:** Potentially, yes. I'd design your pipeline to treat it as an optional feature — include it if cleared, exclude it if not. Don't make it load-bearing for model performance.

**Vishwavani:** *(joining the call late)* Sorry — just catching the last bit. Does that affect the segment filter? Like, can we still score RC Direct customers even if vet referral is excluded?

**Rahul:** Yes, scoring RC Direct customers is fine. It's just that specific field — the vet referral flag and prescription tier — that's under review. All the transactional and engagement features are clear.

**Harshita:** Good. We'll design around that.

**Rahul:** Second thing — we have a data freeze period around Diwali each year, October 1 to November 15 approximately, where no production changes are allowed. If Phase 2 depends on production Snowflake or Airflow changes in that window, plan for it now.

**Harshita:** Noted. Phase 2 is targeting September — so we'd need to be in production well before October 1 to have time for any fixes.

**Rahul:** Exactly.

**Harshita:** Rahul, this has been very precise and useful. Exactly the kind of constraints we need early. I'll send through a summary and flag the open DPA items for follow-up.

**Rahul:** One thing from my side — once DPA is signed, come back to me early to plan the service account setup. We have a two-week provisioning SLA and I'd rather not be the critical path at the end of the project.

**Harshita:** Will do. Thank you.

---

---

## Post-Session Notes (Consolidated)

```
SESSION: W-01
DATE: 31 March 2026
ATTENDEES (Mars side): Priya Sharma — Head of CRM & Retention
ATTENDEES (Mu Sigma side): Harshita Gupta

KEY FINDINGS:
1. Current at-risk process is purely recency-based (45-day threshold for RC Direct,
   ~60-day for Pedigree Club) — delivered as manual Excel every Monday
2. Entire filtering logic lives with one coordinator (Deepa) — single point of failure
3. No holdout group for retention measurement — reported 22% RC retention rate is likely inflated
4. Output must land in Salesforce (not just Snowflake) for CRM team to act on it
5. Need a fourth output tier: "Do Not Contact" suppression for Gold/Platinum

SURPRISES / THINGS WE DID NOT EXPECT:
1. Two different churn thresholds already operating across segments — label
   inconsistency is worse than expected
2. Priya identified "ghost purchasers" as a critical missed segment — customers whose
   low baseline purchase frequency means the 45-day rule never triggers
3. Channel-switching false positives are a known pain point — customers buying via
   Amazon/vet clinic get wrongly flagged

OPEN ITEMS:
1. Formal churn definition — needs cross-functional alignment (CRM + Finance + Analytics)
2. Salesforce integration spec — Phase 1 out vs in scope to be confirmed
3. Deepa to be included in UAT planning

CONFIRMED DECISIONS:
1. Scoring cadence: Weekly, Sunday night, scores available Monday morning
2. Output format: Risk score + 4-tier label (High / Medium / Low / Do Not Contact)
```

```
SESSION: W-02
DATE: 31 March 2026
ATTENDEES (Mars side): Karthik Nair — Analytics & Data Lead
ATTENDEES (Mu Sigma side): Harshita Gupta, Vishwavani

KEY FINDINGS:
1. Salesforce + Oracle in Snowflake via Fivetran. Capillary (loyalty) in Snowflake
   via fragile custom script — no data quality checks
2. 90-day and 12-month aggregations exist in dbt marts layer — full refresh,
   ~90 min runtime. Needs migration to incremental model
3. Prior XGBoost model (Manish, early 2025) had critical feature leakage —
   subscription_active used as feature. Never deployed. Notebooks available
4. Practical training window: 2022 onwards (pre-2022 data has migration gaps)
5. Customer population: ~285K RC Direct active, ~1.1M Pedigree Club active
   (~170K and 650K scoreable after removing very long-inactive)
6. Identity bridge coverage: ~68% RC Direct, ~55% Pedigree Club — a third of
   customers have incomplete cross-system identity

SURPRISES / THINGS WE DID NOT EXPECT:
1. dbt layer already exists — we don't need to build feature aggregations from scratch
2. Freshdesk complaint data exists but production ingestion isn't built —
   num_complaints_12m in sample was a manual export
3. WhatsApp engagement signal is directionally correct but webhook reliability ~90%

OPEN ITEMS:
1. Karthik to share prior XGBoost notebooks
2. Need decision on subscription_active as feature vs label — leakage risk confirmed
3. Freshdesk ingestion pipeline — DE scope item to add for Phase 1
4. Identity bridge improvement — flag records with incomplete coverage in model output

CONFIRMED DECISIONS:
1. Training window: 2022–present
2. Feature aggregations sourced from existing dbt mart, refactored to incremental
3. LightGBM as primary model, Logistic Regression as baseline
```

```
SESSION: W-03
DATE: 31 March 2026
ATTENDEES (Mars side): Rahul Mehta — IT Platform & Infrastructure Lead
ATTENDEES (Mu Sigma side): Harshita Gupta, Savikar

KEY FINDINGS:
1. Snowflake: Enterprise, AWS ap-south-1. Separate non-prod and prod accounts.
   PII masked in non-prod (tokenised IDs, masked email/phone)
2. MWAA Airflow 2.8.1, Python 3.11. LightGBM NOT in current requirements.txt —
   must submit package request (3–5 days turnaround)
3. Production deployment: Change request in ServiceNow + architecture board review
   + 1-week UAT = 2–4 weeks from code complete. Must account for this in timeline
4. DPA: Under review — two clauses (data residency + sub-processor). Est. 3–4 weeks
   to completion = data access earliest early May
5. vet_referred field and prescription-tier fields under InfoSec review for health
   data classification. Treat as optional feature pending ruling (~2–3 weeks)
6. Production change freeze: Oct 1 – Nov 15 annually

SURPRISES / THINGS WE DID NOT EXPECT:
1. Production deployment cycle is 2–4 weeks — this directly compresses the June 30
   deadline. Must reach code-complete by ~June 1 to be safe
2. Sub-processor clause in DPA could have required separate cloud approval —
   confirmed in-Snowflake processing avoids this complication

OPEN ITEMS:
1. DPA completion — track weekly with Rahul and Mars Legal
2. LightGBM package request to submit as soon as DPA is signed
3. Service account provisioning — flag early, 2-week SLA
4. vet_referred classification ruling — expected within 2–3 weeks

CONFIRMED DECISIONS:
1. All training and inference stays within Mars Snowflake — no data leaves to
   external infrastructure
2. Python 3.11, MWAA 2.8.1
3. Snowflake warehouse: X-Small for dev, scaling request process for production
4. DPA-first policy — no data access until paperwork complete
```
