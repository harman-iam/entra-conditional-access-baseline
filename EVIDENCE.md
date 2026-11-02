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

[Sign-in log](initial-policies/05-signin-log-report-only-results.png): real sign-in showing CA003 as "Report-only: User action required", with CA001 and CA002 not applied.

Policy details: [CA001](initial-policies/06-policy-details-ca001-admins-mfa.png), [CA002](initial-policies/07-policy-details-ca002-block-legacy.png), [CA003](initial-policies/08-policy-details-ca003-allusers-mfa.png).
