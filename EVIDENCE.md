# Conditional Access evidence

Lab tenant screenshots. Account, tenant and request identifiers are redacted. All What If results were captured with the tested policies in Report-only.

## Final baseline

| Test | User | Conditions | Expected | Actual |
|---|---|---|---|---|
| [01](final-baseline/01-whatif-alex-browser-applied.png) | alex.employee | Graph Explorer, Windows, Browser | CA200 applies | Match |
| [02](final-baseline/02-whatif-alex-device-code-applied.png) | alex.employee | Same, plus device code flow | CA200 and CA002 (Block) apply | Match |

Final configuration: [policy list](final-baseline/03-policy-list.png), [CA001](final-baseline/04-policy-details-ca001-block-legacy.png), [CA002](final-baseline/05-policy-details-ca002-block-device-code.png), [CA100](final-baseline/06-policy-details-ca100-admins-mfa.png), [CA200](final-baseline/07-policy-details-ca200-internals-mfa.png), [CA400](final-baseline/08-policy-details-ca400-guests-mfa.png).

CA001 and CA100 are On; CA002, CA200 and CA400 are Report-only.

The What If tests were run before the final edits, which did not change the policies that apply to alex.employee. CA100 was not retested with What If after the final edits.

## Initial policies (old names)

| Test | User | Conditions | Expected | Actual |
|---|---|---|---|---|
| [01](initial-policies/01-whatif-alex-modern-auth-applied.png) | alex.employee | Exchange Online, modern auth client | CA003 applies | Match |
| [02](initial-policies/02-whatif-alex-legacy-client-applied.png) | alex.employee | Exchange Online, Exchange ActiveSync | CA002 (Block) and CA003 apply | Match |
| [03](initial-policies/03-whatif-admin-applied.png) | Admin account | Exchange Online, modern auth client | CA001 and CA003 apply | Match |
| [04](initial-policies/04-whatif-breakglass-not-applied.png) | Emergency Access 01 | Exchange Online, modern auth client | No policy applies (Users and groups) | Match |

Risk-based access and risky-user response
Tested in a personal lab tenant on November 3–4, 2026, using a test user. Account, tenant and object identifiers are redacted. Both risk policies stayed in Report-only.
Policies
Policy	Condition	Control
CA201-Internals-AllResources-SignInRisk-RequireMFA	Sign-in risk: Medium, High	Require MFA, sign-in frequency every time
CA202-Internals-AllResources-UserRisk-RequireRemediation	User risk: High	Require risk remediation, sign-in frequency every time
The risk conditions are in separate policies because all conditions in one policy must match. A combined policy would miss a risky sign-in from a user who isn't flagged.
What If results
Test	Result	Evidence
Test user, medium sign-in risk	CA201 applies, CA202 doesn't	01
Test user, high user risk	CA202 applies, CA201 doesn't	02
Test user, both risks high (user field not visible in the screenshot)	Both apply	03
Break-glass account, both risks high (name redacted)	0 policies apply	04
CA200 also applies to the test user in the first three tests, because it covers internal users regardless of risk. What If displays CA202's grant as "Authentication strength: Multifactor authentication". The portal selected this automatically with Require risk remediation.
Response test
Stage	What happened	Evidence
Raise risk	Marked the test user compromised through Microsoft Graph (204 No Content)	05
Confirm	Risky users showed Confirmed compromised, High	06
Remediate	Admin password reset changed the state to Remediated, risk level None	07
Audit	Log shows the reset, session revocation (Update StsRefreshToken entries) and Disable account	08
Scope TAP	TAP was enabled for all users, then scoped to one group. Policy allows 1–8 hours and is multi-use by default	09, 10, 12
Recover	One-time TAP used to register Microsoft Authenticator and a synced passkey. Both were tested with fresh sign-ins	11
Limits of this test
No real detection fired, because the risk was set manually. The Risk detections report wasn't tested.
Report-only means these policies didn't enforce anything on the test user.
Inbox rule checks weren't tested.
The user page refused the password reset for a while, then worked. Risky users > Reset password worked meanwhile. The cause wasn't confirmed.
Key lesson: Remediated doesn't mean safe. A reset can clear the risk flag without removing anything an attacker added, so persistence checks still happen.

[Sign-in log](initial-policies/05-signin-log-report-only-results.png): real sign-in showing CA003 as "Report-only: User action required", with CA001 and CA002 not applied.

Policy details: [CA001](initial-policies/06-policy-details-ca001-admins-mfa.png), [CA002](initial-policies/07-policy-details-ca002-block-legacy.png), [CA003](initial-policies/08-policy-details-ca003-allusers-mfa.png).
