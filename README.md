# Relève: deployment notes

## What's here
- `index.html`: the whole website and app (marketing site, physician app, clinic app, admin verification). No build step.
- It connects to the Supabase project **releve-locum** (region ca-central-1, Montréal) using the public anon key. Access is enforced in the database by row-level security, so that key is safe to ship in the page.
- If the database can't be reached, the page falls back to sample data and says so in a banner.

## Put it online (about 5 minutes)
1. Go to https://app.netlify.com/drop and drag this folder onto the page. (Vercel or Cloudflare Pages also work: deploy the folder as a static site.)
2. In Supabase: open the project, then **Authentication > URL Configuration**:
   - Set **Site URL** to your new address (e.g. https://releve.netlify.app or your own domain).
   - Add the same address under **Redirect URLs**.
   Without this, sign-up confirmation emails link to localhost.
3. In **Authentication > Providers > Email**, keep "Confirm email" on.
4. In **Authentication > Attack Protection** (Pro plan), turn on leaked-password protection.

## Colours
- Canadian red: FIP red (Pantone 032, RGB 235 45 55, #EB2D37) for the logo mark and the red rule at the top of every page; a deeper red #C8102E for buttons and links, so white text on it passes WCAG AA contrast.
- Navy #1B2A3A for text, with snow #F4F6F8 and glacier #E6EDF2 neutrals.
- The site deliberately does **not** use the Canadian flag, the Canada wordmark or the Government of Canada page layout. Those are official federal marks, and a private company using them could look like a government service.

## Photos (licensed)
| Use | Photo | Photographer | Licence |
|---|---|---|---|
| Hero | Winter highway, Canada | Myriam Olmz | Unsplash License, https://unsplash.com/photos/U2o5rjrSr2Q |
| Coast strip | Peggy's Cove, Nova Scotia | matthew smith | Unsplash License, https://unsplash.com/photos/-cDhdCEJryI |
| Coast strip | Montréal, Québec | Etienne Delorieux | Unsplash License, https://unsplash.com/photos/r-XORSP-t2I |
| Coast strip, sign-in | Chilliwack, British Columbia | Megan te Boekhorst | Unsplash License, https://unsplash.com/photos/6almqMlQgxA |
| Clinics section | Stethoscope and laptop | Negative Space | Pexels License, https://www.pexels.com/photo/48604/ |

Both licences allow free commercial use with no attribution required (credits are shown anyway). You can't resell the photos as-is or imply that anyone pictured endorses Relève.
The site currently loads them from Unsplash and Pexels. For production, download each one at full size into an `img/` folder and change the `src` values in the `PH` list near the top of the script, so the site doesn't depend on third-party hosts.

## Demo accounts (sample data)
All use the password you were given in the chat. Delete them before real launch
(Authentication > Users, filter by `demo.releve.ca`).

| Email | Role |
|---|---|
| amara.okafor@demo.releve.ca | Physician (BC + ON licences) |
| jl.tremblay@demo.releve.ca | Physician (QC + ON) |
| priya.raman@demo.releve.ca | Physician (ON) |
| ethan.macleod@demo.releve.ca | Physician (NS, BC, AB) |
| sofia.mendes@demo.releve.ca | Physician (AB) |
| clinic@demo.releve.ca | Clinic scheduler, Danforth Family Health Team |
| regional@demo.releve.ca | Clinic scheduler, 7 other clinics |
| admin@demo.releve.ca | Platform admin (credential verification) |

## Backend summary
- Tables: profiles, clinicians, clinics, clinic_members, credentials, shifts, staff_bank, invitations, bookings, timesheets, invoices, account_actions, deletion_requests, audit_log, provinces, app_settings.
- Actions (database functions): shift_board, book_shift, confirm_held_booking, cancel_booking, submit_timesheet, approve_timesheet, dispute_timesheet, mark_invoice_paid, rank_candidates, invite_clinician, decline_invitation, issue_account_action, appeal_account_action, decide_appeal, export_my_data, request_account_deletion.
- Nightly job (07:15 UTC): expires lapsed credentials, applies due account actions, closes finished shifts.
- Private storage bucket `credential-docs`, one folder per physician.
- Settings to review before launch (table `app_settings`): early_pay_fee_pct (currently 0), payment_terms_days (14), late_interest_pct_month (2), removal_notice_days (7).

## Before taking real users
- Legal review of the locum agreement wording, privacy notice and terms (Canadian health-privacy and employment counsel).
- Québec Law 25 privacy impact assessment, privacy officer named on the site.
- Credential verification is manual (admin screen). Automating checks against college registers needs agreements with each college.
- Payments: invoices are generated, but money doesn't move yet. Connect a Canadian payment processor for payouts and early pay.
- Email notifications (new invitation, timesheet approved, expiry reminders) are not built yet.
