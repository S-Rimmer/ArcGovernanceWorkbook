# Azure Arc Governance Dashboard (Azure Monitor Workbook)

A tenant-portable Azure Monitor Workbook that provides a governance view for **Azure Arc-enabled servers**. It surfaces Arc connectivity, Connected Machine Agent version, automatic upgrade status, Defender for Cloud plan status, Microsoft Defender for Endpoint (MDE) extension coverage, Azure Policy compliance, Windows licensing / Azure Hybrid Benefit (AHB) indicators, and Arc-enabled SQL Server license status.

> **Design note:** The queries deliberately avoid complex joins so the workbook renders safely even when Defender for Cloud, MDE, Arc SQL, or AHB are not deployed. Sections simply return empty or "Not Reported" instead of erroring.

---

## Contents

- `ArcGovernanceWorkbook.json` — the workbook definition (Azure Monitor Workbooks `Notebook/1.0` format).

---

## How to import

1. In the Azure portal, go to **Monitor** > **Workbooks**.
2. Select **+ New**.
3. Select the **Advanced Editor** button (`</>`).
4. Delete the placeholder content, paste the full contents of `ArcGovernanceWorkbook.json`, and select **Apply**.
5. Confirm the scope at the top of the workbook (see [Scope](#scope) below).
6. Select **Save**, then provide a name, resource group, and region to store the workbook.

You can also import it as a shared/gallery template — paste the JSON in the same Advanced Editor.

---

## Scope

The workbook and every query are scoped to **`value::all`**, which resolves to **all subscriptions the signed-in user can access** in the current tenant. There are no hardcoded subscription IDs, so it works in any tenant without edits.

To narrow the view, use the subscription/scope picker at the top of the workbook after import.

---

## Required permissions

Data is read through **Azure Resource Graph**. The user viewing the workbook needs:

| Capability | Role |
|---|---|
| Read Arc servers, extensions, licensing (`resources`) | **Reader** on the target subscriptions |
| Defender for Cloud plan status (`securityresources`) | **Security Reader** |
| Azure Policy compliance (`policyresources`) | Reader (Policy Insights read) |
| Save the workbook | **Monitoring Contributor** (or workbook save rights) in the target resource group |

Data populates only where the corresponding resources exist (Arc servers, Defender for Servers, MDE extensions, Arc-enabled SQL). Empty sections are expected otherwise.

---

## Sections

| # | Section | What it shows |
|---|---------|---------------|
| — | Executive Summary | Totals: Arc servers, connected vs. disconnected, auto-upgrade enabled/disabled, overall health. |
| 1 | Arc Server Inventory | Per-server connectivity, agent version, and auto-upgrade status with color/icon indicators. |
| 2 | Arc Agent Version Distribution | Count of servers by installed Connected Machine Agent version. |
| 3 | Auto Upgrade Compliance | Pie of Enabled / Disabled / Not Reported auto-upgrade. Recommended control instead of tracking latest agent version manually. |
| 4 | Defender for Cloud Plan Status | Defender for Servers pricing tier (Standard/Free) per subscription. |
| 5 | MDE Extension Coverage | Presence of `MDE.Windows` / `MDE.Linux` Arc extensions, plus an extension inventory. |
| 6 | Azure Policy Compliance | Compliance states for Arc-enabled servers, grouped by policy assignment/definition. |
| 7 | Windows Server AHB / Licensing | Windows licensing, subscription, Software Assurance, and ESU indicators. |
| 8 | Arc-enabled SQL Server License Status | SQL license type (AHB / PAYG) for Arc-enabled SQL instances. |
| 9 | Governance Scorecard | Single-row roll-up of Arc connectivity and auto-upgrade health. |

---

## Notes

- Built for **Azure Resource Graph** (`resourceType: microsoft.resourcegraph/resources`); no Log Analytics workspace is required.
- Auto-upgrade "Disabled / Not Reported" is treated as the primary operational risk, so the workbook does not need monthly edits to track the newest agent build.
- The `$schema` reference is informational; the portal ignores it on import.
