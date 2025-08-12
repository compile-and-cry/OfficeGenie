# New Joiner Assistant Flow Blueprint

This blueprint describes the end-to-end Power Virtual Agents topics and Power Automate flows for the New Joiner Assistant (AI FAQ, Document Navigator, Quiz, Gamification, and Progress Tracking).

## A. Power Virtual Agents Topics

1. **AI FAQ Bot**
   - Connect a PVA topic to the SharePoint FAQ list using the *List Records* action.
   - (Optional) Add an Azure OpenAI call to answer FAQs conversationally.

2. **Document Navigator**
   - PVA topic queries SharePoint Document Library or list of onboarding resources.
   - Returns link attachments via Adaptive Cards.

3. **Onboarding Checklist & Progress Tracker**
   - Trigger PVA to list tasks from the Onboarding Tasks SharePoint list.
   - Use an Adaptive Card that shows each task name, due date, and status.
   - Allow user to mark tasks done via Action.Submit and invoke Power Automate.

4. **Interactive Quiz**
   - PVA topic uses the Quiz Adaptive Card JSON template.
   - Pull a quiz row from Excel and populate `Question` and `choices` dynamically.
   - Post Adaptive Card (using `templates/quiz-adaptive-card.json`) and wait for response.
   - The user’s answer is returned to the flow for scoring.

## B. Power Automate Flow Components

### 1. Award Points & Badges
#### 1. Award Points & Badges Flow

This flow awards points (and optional badges) when a user completes a task or answers a quiz correctly.

**Step-by-Step Implementation:**

1. **Create the flow**
   - In Power Automate, click **Create** → **Automated cloud flow**.
   - Name it **Award Points Flow** and select trigger **When an item is modified** (SharePoint).

2. **Configure trigger condition (tasks only)**
   - Open trigger settings (··· → Settings) and under **Trigger Conditions** add:
     ```
     @equals(triggerBody()?['Status'], 'Done')
     ```

3. **Fetch task item and current points**
   - Add **Get item** (SharePoint): use same Site Address and List Name, set **Id** to **ID** from trigger.
   - Add **Initialize variable** (`currentPoints`, Type: Integer) with value **Points** (from Get item).

4. **Calculate new points**
   - Add **Compose** (rename to **addPoints**) with **Points** from the trigger (task row).
   - Add **Compose** (rename to **newPoints**) with expression:
     ```
     add(variables('currentPoints'), outputs('addPoints'))
     ```

5. **Update task item**
   - Add **Update item** (SharePoint) and set **Points** field to **Outputs** of **newPoints**.

6. **Handle quiz submissions (parallel branch)**
   - Create a branch triggered by **When a new row is added** (Excel Online): Quizzes table in OneDrive.
   - Add a **Condition** to compare **Submitted Answer** to **Correct Option**.
   - Under **If yes**, replicate steps 3–5 using the quiz row’s Points value.

7. **Optional: Notify user in Teams**
   - After updating points, add **Post a message (V3)** (Microsoft Teams) to congratulate the user and show their new point total.

8. **Return data to PVA**
   - At the end of the flow, use **Respond to Power Virtual Agents** action to send success/failure status back to the bot.

### 2. Progress Update Notification
1. **Trigger**: scheduled flow (Daily or weekly).
2. **Action**: query Onboarding Tasks list and count completed vs pending.
3. **Post**: send an Adaptive Card summary to the user or HR channel with progress stats.

### 3. Leaderboard Dashboard Refresh
1. **Trigger**: after points update or scheduled (e.g., hourly).
2. **Action**: refresh Power BI dataset or rebuild Adaptive Card leaderboard.

## C. End-to-End Sequence
1. User chats with PVA: asks a question → FAQ Bot replies.
2. User requests document links → Document Navigator posts resource cards.
3. User starts quiz → PVA posts Adaptive Card → user submits answer → Flow awards points.
4. User marks task done → Flow awards task points and updates progress.
5. Scheduled flow posts progress summary and updated leaderboard.

This blueprint covers all PRD features for the New Joiner Assistant.
