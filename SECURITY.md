# Security policy — SCIENCE QUEST MAX

## Before public launch
- Set `NODE_ENV=production` and a unique `ADMIN_KEY` with at least 32 cryptographically random characters. The server refuses to start in production without it.
- Use HTTPS and a trusted reverse proxy. Set `TRUST_PROXY=1` only when the hosting proxy reliably overwrites/appends forwarding headers.
- Restrict administrator access; rotate the key if it may have been exposed. Never place it in frontend HTML, source control, screenshots, or issue reports.
- Use persistent storage and tested backups. The current JSON database and in-memory login/quiz sessions are single-instance starter components, not horizontally scalable production storage.
- Monitor errors, request volume, disk space, account abuse, and failed logins. Set alerts and rehearse restore procedures.
- Do not collect personal information that the service does not need. Publish a reviewed privacy policy and terms before accepting public users.

## Reporting a vulnerability
Do not publish secrets or exploit details in public issues. Contact the service operator privately with reproduction steps and impact. This repository does not currently designate a monitored security contact.

## Scope and limitations
Rate limiting is process-local and may be bypassed across multiple instances. User sessions and quiz sessions are held in memory and are lost on restart. This project has not undergone an independent penetration test or load test.
