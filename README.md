# Zero Trust Conditional Access Baseline

Oct 31, 2026 · Harman

Test results and screenshots are in [EVIDENCE.md](EVIDENCE.md).

## Purpose

This baseline protects every sign-in to a Microsoft Entra ID tenant with 10 Conditional Access policies, grouped by persona instead of by request.

Each persona has a defined risk and a matching set of controls, so coverage is predictable and gaps are easy to spot. The design follows the persona model in Microsoft's Conditional Access framework and is built in a personal lab tenant with Entra ID P2.

## Personas and numbering

Policies are numbered in persona ranges so each persona's policies sort together and new ones can be added without renaming.

| Range | Persona | Who it covers | Main risk |
| --- | --- | --- | --- |
| CA000–099 | Global | Everyone in the tenant | Weak sign-in paths that bypass MFA |
| CA100–199 | Admins | Holders of privileged directory roles | One compromised account controls the tenant |
| CA200–299 | Internals | Employees | Phishing and password reuse |
| CA300–399 | Externals | Contractors with member accounts in this tenant | Unmanaged devices; reserved, not used in this lab |
| CA400–499 | Guests | B2B users from other tenants | Security controlled by another organization |
| CA900–999 | Workload identities | Single-tenant service principals | Leaked secrets; cannot do MFA |

Ranges 500–899 are reserved for further personas such as guest admins and service accounts.

## Policy baseline

All 10 policies exclude CA-Exclude-BreakGlass. Five are built in the lab; five are design-only because they need Intune, a separate licence, or later weeks.

| Policy | Users | Target resources | Conditions | Control |
| --- | --- | --- | --- | --- |
| CA001-Global-AllResources-BlockLegacyAuth | All users | All resources | Client apps: Exchange ActiveSync, other clients | Block |
| CA002-Global-AllResources-BlockDeviceCodeFlow | All users | All resources | Authentication flows: device code flow | Block |
| CA100-Admins-AllResources-RequirePhishingResistantMFA | Privileged directory roles | All resources | None | Authentication strength: phishing-resistant MFA |
| CA101-Admins-AllResources-SignInFrequency4h | Privileged directory roles | All resources | None | Session: sign-in frequency 4 hours, no persistent browser |
| CA102-Admins-AdminPortals-RequireCompliantDevice | Privileged directory roles | Microsoft admin portals | None | Require compliant device |
| CA200-Internals-AllResources-RequireMFA | All users, excluding guests | All resources | None | Require MFA |
| CA201-Internals-AllResources-BlockHighSignInRisk | All users, excluding guests | All resources | Sign-in risk: high | Block |
| CA400-Guests-AllResources-RequireMFA | Guest or external users | All resources | None | Require MFA |
| CA401-Guests-NonApprovedApps-Block | Guest or external users | All resources except approved apps | None | Block |
| CA900-WorkloadIdentities-AllResources-BlockUntrustedLocations | Single-tenant service principals | All resources | Locations: any except trusted named locations | Block |

## Why each policy exists

| Policy | Why |
| --- | --- |
| CA001 | Legacy protocols are not able to do MFA, so they're the main path for password spray attacks. To avoid these scenarios we setup this policy so no sign in from legacy authentication can get in. |
| CA002 | MFA confirms the real user approved the sign-in, but it can’t tell whether they were tricked into approving someone else’s device. An attacker can request a code on their own device and persuade a user to enter it on the real Microsoft site. The user passes MFA, and the attacker gets the access. We block this flow because most users don’t need it, and we allow exceptions only for a specific account or device, with a documented reason and a review date. |
| CA100 | Admin accounts are the highest-value target. Phishing-resistant methods are bound to the real domain, so AiTM kits can't relay them. |
| CA101 | A stolen admin session is limited to 4 hours, and admins don't stay signed in on shared browsers. |
| CA102 | Admin portals are only reachable from managed, healthy devices, so a stolen password alone isn't enough. |
| CA200 | MFA for every employee blocks most password-based attacks. Guests are excluded because CA400 covers them. |
| CA201 | Identity Protection flags sign-ins that look like an attack, such as anonymous IPs or impossible travel. High risk is blocked outright. |
| CA400 | Guest security depends on another organization. MFA ensures a stolen guest password isn't enough; cross-tenant trust can avoid double prompts. |
| CA401 | Guests only reach the apps they were invited for, which limits damage if a guest account is compromised. |
| CA900 | Service principals can't do MFA. Restricting them to known IPs means a leaked secret is useless from anywhere else. |

## Design principles

- **Target groups and roles, never individual users.** New joiners and role changes pick up the right policies automatically.
- **Every exclusion is a group with an owner, a reason and a review.** Temporary exclusions have an expiry date and go through change management.
- **Break-glass accounts:** two cloud-only accounts on the .onmicrosoft.com domain, permanent Global Administrator (not PIM-eligible), excluded from every policy, protected with MFA (Microsoft Authenticator in the lab, FIDO2 as the target design), and any sign-in triggers an alert.
- **Names describe the policy:** CA\[number\]-\[Persona\]-\[Target resources\]-\[Control\]-\[Condition\], so anyone can read a policy's purpose from the list.
- **Block wins.** Every matching policy applies, so block policies are kept narrow and tested first.

## Rollout and lab status

Every new policy starts in report-only mode, is tested with the What If tool and checked in the sign-in logs, then switched On one persona at a time.

| Policy | Lab state | Notes |
| --- | --- | --- |
| CA001 | On | Renamed from CA002 |
| CA002 | Report-only | New; tested with What If |
| CA100 | On | Renamed from CA001; uses Require MFA until a passkey is registered (Week 5) |
| CA101 | Design only |  |
| CA102 | Design only | Needs Intune device compliance |
| CA200 | Report-only | Renamed from CA003; guests excluded |
| CA201 | Design only | Built in Week 5 with Identity Protection |
| CA400 | Report-only | New; built in report-only |
| CA401 | Design only | Needs approved guest apps (Week 11) |
| CA900 | Design only | Needs Workload Identities Premium licence |

Before the P2 trial ends around November 30, 2026, enforced policies go back to report-only or off.

## Next steps

- [x] Rename CA001, CA002 and CA003 to the new names
- [x] Build CA002 and CA400 in report-only and test with What If
- [ ] Register a passkey, then switch CA100 to the phishing-resistant strength (Week 5)
- [x] Register MFA on both break-glass accounts
- [x] Review the "Why" column before publishing to GitHub
- [ ] Codify this baseline in Terraform (Week 20)
