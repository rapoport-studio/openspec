# Closure email template

> **Usage:** Send to the client's primary contact at engagement off-ramp. Fill in all `<placeholder>` slots before sending. The Appendix C checklist below is taken verbatim from `_methodology/rapoport-studio-engagement.md` — keep them in sync.

---

**Subject:** Closing our engagement — [Engagement name]

Hi [Client primary contact],

We're ready to close [Engagement name]. Per § 11 of our engagement charter,
closure runs through a structured checklist so neither side has to remember
the small things.

The checklist below is taken directly from Appendix C of our charter:

---

### C.1 Spec & artifacts (4)
- [ ] OpenSpec triplet merged into client `main` branch — link: <URL>
- [ ] Linear issues exported to client-owned destination — format: <CSV | JSON>
- [ ] Canvas conversation history snapshot offered — format: <Markdown export>
- [ ] Mirror perception document in client repo — path: <repo path>

### C.2 Access revocation (4)
- [ ] Studio-member access to client repo revoked — confirmed by: <client-side reviewer>
- [ ] Specialist canvas access revoked — confirmed by: <studio>
- [ ] Bot tokens / OAuth grants revoked — confirmed by: <client>
- [ ] Telegram / WhatsApp surface archived or transferred — disposition: <archived | transferred>

### C.3 Knowledge transfer (2)
- [ ] Final review meeting completed — date: <YYYY-MM-DD>
- [ ] Postmortem published or scheduled — link: <URL> | scheduled-for: <YYYY-MM-DD>

### C.4 Financial & consent (2)
- [ ] Final invoice settled
- [ ] Case study consent confirmed (or declined or deferred) — disposition: <consent | declined | deferred>

---

If anything on the list is incomplete or unclear, reply here and we'll
hold closure until it's resolved.

The full charter (and all our methodology) lives publicly at
https://github.com/rapoport-studio/openspec-public — if you ever want to
refer back to how this worked.

— [Engagement lead]
