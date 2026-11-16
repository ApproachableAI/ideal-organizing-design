# Live lead-capture rules in Streak (NEW PIPELINE)

Runbook for the two Streak automations that turn website-form and Calendly emails in info@idealorganizers.com into boxes in NEW PIPELINE. Built November 13–14, 2026, live-tested and passed November 16, 2026. These rules live in Streak (Integrations & Automation › Mine, owned by info@); this file is the reference copy. If you edit a rule in Streak, update this file.

## Rule 1: "LIVE – Website form email → NEW PIPELINE"

- **Trigger:** Google › Incoming message, ignore replies.
  Filter: `from:notification@ccmsites.com subject:("New Client Form" OR "New message from" OR "INTERIOR DESIGN FORM Submission")`
- **Steps:**
  1. Create a box. Pipeline NEW PIPELINE. Name `Website lead – {{ incomingMessage.subject }} – {{ incomingMessage.gmailThreadId }}` (the thread ID keeps names unique; Streak reuses a box with the same name). Stage "New Lead / Initial Outreach Sent".
  2. Add Gmail thread to box.
  3. Update a box: Lead Source = "Contact us form on site"; Inquiry Date = the Inquiry Date formula below.
  4. AI Autofill (box key `=addThreadToBox.boxKey`): Caller Phone, Client Type, Lead Summary, How did you find us. Skip filled cells.
  5. Add task "Call this lead" (in hours), assigned to info@. Should run? = in-hours formula. Due `in 1 hour`.
  6. Add task "Call this lead" (after hours), assigned to info@. Should run? = after-hours formula. Due = next-business-day formula.

## Rule 2: "LIVE – Calendly booking email → NEW PIPELINE"

- **Trigger:** Google › Incoming message, ignore replies. Filter: `from:notifications@calendly.com subject:"New Event"`
- **Steps:**
  1. Create a box. Pipeline NEW PIPELINE. Name `Calendly – {{ incomingMessage.subject }}`. Stage "Consultation Scheduled/Done". Lead Source "Calendly Booking".
  2. Add Gmail thread to box.
  3. Update a box: Inquiry Date = the Inquiry Date formula below.
  4. AI Autofill (box key `=addThreadToBox.boxKey`): Caller Phone, Lead Summary, How did you find us, Meeting Date. Skip filled cells.
  5–6. The same two task steps as Rule 1.
- No cancellation rule: cancelled bookings are moved by hand.

## Formulas

Streak formulas run in UTC. `etParts` converts a moment to US Eastern time: offset 4 hours from the second Sunday in March at 07:00 UTC until the first Sunday in November at 06:00 UTC, otherwise 5. Checked by script against real America/New_York time every 15 minutes from 2026 through 2028 with zero mismatches (November 14, 2026). Holidays are not skipped.

**In-hours (Should run? on the "in 1 hour" task):** Mon–Fri, 9:00 to 16:59 Eastern.

```
{{ (function(){ function etParts(t){ var y=new Date(t).getUTCFullYear(); var m8=new Date(Date.UTC(y,2,8)).getUTCDay(); var s=Date.UTC(y,2,8+(7-m8)%7,7); var n1=new Date(Date.UTC(y,10,1)).getUTCDay(); var e=Date.UTC(y,10,1+(7-n1)%7,6); var off=(t>=s&&t<e)?4:5; return {d:new Date(t-off*3600000), off:off}; } var p=etParts(Date.now()); var dy=p.d.getUTCDay(); var h=p.d.getUTCHours(); return dy>=1&&dy<=5&&h>=9&&h<17; })() }}
```

**After-hours (Should run? on the second task):** the same formula with `!` in front.

```
{{ !(function(){ function etParts(t){ var y=new Date(t).getUTCFullYear(); var m8=new Date(Date.UTC(y,2,8)).getUTCDay(); var s=Date.UTC(y,2,8+(7-m8)%7,7); var n1=new Date(Date.UTC(y,10,1)).getUTCDay(); var e=Date.UTC(y,10,1+(7-n1)%7,6); var off=(t>=s&&t<e)?4:5; return {d:new Date(t-off*3600000), off:off}; } var p=etParts(Date.now()); var dy=p.d.getUTCDay(); var h=p.d.getUTCHours(); return dy>=1&&dy<=5&&h>=9&&h<17; })() }}
```

**After-hours due date:** the next 9:00 AM Eastern on a weekday (the same day if the lead arrives before 9 AM on a weekday).

```
{{ (function(){ function etParts(t){ var y=new Date(t).getUTCFullYear(); var m8=new Date(Date.UTC(y,2,8)).getUTCDay(); var s=Date.UTC(y,2,8+(7-m8)%7,7); var n1=new Date(Date.UTC(y,10,1)).getUTCDay(); var e=Date.UTC(y,10,1+(7-n1)%7,6); var off=(t>=s&&t<e)?4:5; return {d:new Date(t-off*3600000), off:off}; } var p=etParts(Date.now()); var d=p.d; var dy=d.getUTCDay(); var h=d.getUTCHours(); var add=(dy>=1&&dy<=5&&h<9)?0:1; var c=new Date(Date.UTC(d.getUTCFullYear(),d.getUTCMonth(),d.getUTCDate()+add)); while(c.getUTCDay()==0||c.getUTCDay()==6){ c=new Date(Date.UTC(c.getUTCFullYear(),c.getUTCMonth(),c.getUTCDate()+1)); } var off=etParts(Date.UTC(c.getUTCFullYear(),c.getUTCMonth(),c.getUTCDate(),14)).off; return new Date(Date.UTC(c.getUTCFullYear(),c.getUTCMonth(),c.getUTCDate(),9+off)).toISOString(); })() }}
```

**Inquiry Date:** noon Eastern on the day the lead arrived, so the date reads the same in Eastern, Mountain, Hawaii and UTC. A plain `new Date().toISOString()` dates leads after 7 PM Eastern (8 PM in summer) as the next day.

```
{{ (function(){ function etParts(t){ var y=new Date(t).getUTCFullYear(); var m8=new Date(Date.UTC(y,2,8)).getUTCDay(); var s=Date.UTC(y,2,8+(7-m8)%7,7); var n1=new Date(Date.UTC(y,10,1)).getUTCDay(); var e=Date.UTC(y,10,1+(7-n1)%7,6); var off=(t>=s&&t<e)?4:5; return {d:new Date(t-off*3600000), off:off}; } var p=etParts(Date.now()); var d=p.d; return new Date(Date.UTC(d.getUTCFullYear(),d.getUTCMonth(),d.getUTCDate(),12+p.off)).toISOString(); })() }}
```

## Known limits

- The rules watch info@ only. Lead emails sent to another inbox are not captured.
- Boxes are created without a Streak contact, so Allo won't recognize a website or Calendly lead who calls later until someone adds them as a contact.
- AI Autofill costs about 3–4 Streak AI credits per lead against 150 a month on Pro+.
- The live test on November 16 ran only the in-hours path and a daytime Inquiry Date; the first real after-hours lead confirms the rest.
