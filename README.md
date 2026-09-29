# AI-Powered Resume Screening Pipeline

An n8n workflow that screens a batch of resumes against a job description, using a local AI model — no API costs, runs on my own machine.

## Why I built this

Recruiters spend a lot of time manually going through resumes, and it's easy to get fooled by someone who stuffs their resume with keywords but doesn't actually have the experience. I wanted to automate that first-pass screening — but most AI tools either cost a lot in API fees, or if you run them locally to save money, your computer can't handle it well.

So this does both: it's free to run (local model, not a paid API), but I built it carefully so it doesn't crash or slow my laptop to a crawl.

## How it actually works

- **The job description only gets read once per batch, not once per candidate.** It's the same JD for every resume in that run, so there's no reason to re-process it 50 times if I'm screening 50 resumes. That alone cuts down a good chunk of unnecessary AI calls.
- **Before any AI touches a resume, I check if the file is even readable.** Some PDFs are just scanned images with no real text in them. If a resume comes back basically empty, it gets logged separately instead of wasting an AI call trying to grade nothing.
- **Grading happens in two steps, using two AI calls per candidate.** First, one AI call turns the messy resume text into clean, structured data (their experience, education, skills, etc.). Then a second AI call acts like a hiring manager and grades that structured data against the job description — and specifically checks whether someone who *lists* a skill actually shows they've *used* it, so keyword-stuffing doesn't work.
- **The AI's output isn't always clean.** Sometimes it wraps its answer in extra formatting, or the response gets cut off. I wrote code that catches that and cleans it up so one messy response doesn't break the whole run.

## The part I'm actually proud of

When I first got this working, I went back and tried to break it on purpose — and found a real problem: if something went wrong with the AI (a bad response, a call that failed), the pipeline was quietly treating that as if the candidate genuinely scored a 0 and was a "Weak Fit." There was no way to tell the difference between "this person isn't a good fit" and "something technically broke." That's a real risk — a good candidate could get auto-rejected for the wrong reason, and nobody would know to check.

So I fixed that. Now, if something fails, it doesn't get a fake score — it gets flagged separately for manual review, with a note on *why* it failed, and it gets sent to me over Telegram so I actually see it. And if I already know the resume didn't come through properly, I skip the grading step entirely instead of wasting an AI call grading empty data.

I also made sure one bad resume in a batch doesn't stop the whole thing — if candidate #23 out of 50 has a problem, #24 through #50 still get processed. It's flagged, and the batch keeps going.

## How it flows, start to finish

1. Read all the resumes and the job description.
2. Filter out anything unreadable before it reaches the AI.
3. Go through the resumes one at a time (kept sequential on purpose — running things in parallel on my local hardware would overload it).
4. For each one: extract their info, grade it against the job, and log the result — either a real score and verdict, or a manual-review flag if something went wrong.
5. Everything lands in a Google Sheet, with a Telegram alert for anything that needs a human to look at it.

## What I'm using

- **n8n** to build and run the whole workflow
- **Ollama**, running a local model, so there's no per-call API cost
- **Google Sheets** and **Telegram** for output and alerts

## What I'd still like to improve

- A 7B parameter local model has real limits under heavier load — larger batches or longer resumes push it toward the truncation and context-window issues mentioned above, so this isn't yet built to scale past a local, moderate-volume use case
- Right now, running the same batch of resumes twice would create duplicate entries instead of recognizing repeats — I'd want to fix that with some kind of duplicate check.
- There's a narrow edge case where, if the AI's response is technically valid but happens to be missing the score itself, it could still slip through as a 0 instead of getting flagged — Need to work on this.
- I made unreadable PDFs log to a separate sheet instead of also pinging me on Telegram — on purpose, since as the person using this, I'd rather check that log once in a while than get interrupted for every unreadable file.
