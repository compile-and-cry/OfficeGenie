# OfficeGenie No-Code / Low-Code Implementation Guide

This guide walks through how to strip **OfficeGenie** down to a no-code / low-code build so even a non-coder can set it up in 1–2 weeks using just **Microsoft Teams + Power Platform**. This approach avoids Azure Functions, custom APIs, or complex AI orchestration.

## Platform Stack

* **Microsoft Teams** &ndash; Main user interface (bot, tabs, messages).
* **Power Virtual Agents (PVA)** &ndash; Chatbot logic (FAQ, onboarding steps, inventory queries).
* **SharePoint Lists / Excel in OneDrive** &ndash; Store FAQs, quizzes, onboarding tasks, and inventory data.
* **Power Automate** &ndash; Workflows (alerts, reminders, points calculation).
* **Power BI** &ndash; Leaderboard and inventory dashboard.
* **Microsoft 365 Copilot / Azure OpenAI via PVA** &ndash; (Optional) Conversational AI for FAQ answers.

## Data Storage (No Code)

1. **FAQ List (SharePoint List)**
   * Columns: Question, Answer, Category, Doc Link.
   * Example: "How many leave days do I get?" → "24 per year" → Category = HR → Link = leave-policy.pdf

2. **Quizzes (Excel in OneDrive)**
   * Columns: Question, Option A, Option B, Option C, Correct Option, Points.

3. **Onboarding Tasks (SharePoint List)**
   * Columns: Task Name, Due Date, Status (Pending/Done), Points.

4. **Inventory Usage Log (Excel in OneDrive)**
   * Columns: Item Name, Stock Level, Reorder Threshold, Last Updated, Quantity Used Per Week.

## How It Works (Step-by-Step)

### 1. New Joiner Assistant

* **Power Virtual Agents chatbot** with topics:
  * **FAQs** &ndash; Reads from SharePoint FAQ list.
  * **Onboarding Checklist** &ndash; Shows tasks from SharePoint.
  * **Quiz** &ndash; Fetches a row from Excel and uses Adaptive Cards to ask questions.
* **Gamification**:
  * Power Automate flow triggers when a task or quiz is completed and awards points in SharePoint.
* **Leaderboard**:
  * Power BI reads the Points column from SharePoint and publishes a leaderboard chart in a Teams tab.

### 2. Inventory Forecast Assistant

* **Data Input**:
  * Operations team updates the Excel log weekly (item, current stock, threshold, usage).
* **Forecast**:
  * Use Excel’s built-in FORECAST.ETS to predict next run-out date and save in a new column.
* **Alerts**:
  * Power Automate scheduled flow checks if Stock Level ≤ Threshold and sends an Adaptive Card in Teams to reorder.
* **Dashboard**:
  * Power BI reads the Excel and displays usage trends and predicted run-out dates in a Teams tab.

## Build Order for MVP (1–2 Weeks)

| Days      | Tasks                                                                 |
|-----------|-----------------------------------------------------------------------|
| Day 1–2   | Create SharePoint lists (FAQ, Onboarding Tasks) and Excel files (Quizzes, Inventory). |
| Day 3–5   | Build PVA chatbot: FAQ, onboarding checklist, and quiz topics.        |
| Day 6–8   | Create Power Automate flows for awarding points and low-stock alerts. |
| Day 9–12  | Develop Power BI leaderboard & inventory dashboard; embed in Teams tabs. |
| Day 13–14 | Pilot testing with HR group and new joiners.                         |

## Power Automate Flows

Below are the two key Power Automate flows to wire up gamification and low-stock alerts.

### A. Award Points Flow

1. **Trigger**:
   - For tasks: "When an item is modified" on the Onboarding Tasks SharePoint list.
   - For quizzes: "When a new response is submitted" on the Quizzes Excel table in OneDrive.
2. **Condition**:
   - Tasks: Status equals "Done".
   - Quizzes: Submitted answer equals the Correct Option column.
3. **Actions**:
   - Get the current Points value from the record.
   - Calculate `NewPoints = CurrentPoints + Points` (from the task or quiz row).
   - Update the record (SharePoint item or Excel row) to set Points = NewPoints.
4. **Post-back** (optional): Send a Teams message or bot notification congratulating the user.

### B. Low-Stock Alert Flow

1. **Trigger**: Recurrence (e.g. daily at 8:00 AM).
2. **Action**: "List rows present in a table" on the Inventory Usage Excel file.
3. **Apply to each** row:
   - **Condition**: Stock Level less than or equal to Reorder Threshold.
   - **If yes**: Post an Adaptive Card to a Teams channel (or DM) with item name, current stock, and a reorder link.

These flows can be built drag-and-drop in Power Automate, using built-in connectors for SharePoint, Excel, and Teams.

For detailed, step-by-step UI guidance, see:

- [Award Points Flow Implementation Details](flows/award-points-flow-details.md)
- [Low-Stock Alert Flow Implementation Details](flows/low-stock-alert-flow-details.md)

## Why This Is No-Code Friendly

* No custom code &ndash; Uses out-of-the-box Microsoft 365 tools.
* Editable data &ndash; HR/Ops can update Excel/SharePoint without touching chatbot logic.
* Drag-and-drop flows &ndash; Build workflows visually in Power Automate.
* Visual dashboards &ndash; Create reports in Power BI and embed directly in Teams.
