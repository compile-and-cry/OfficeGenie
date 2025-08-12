# Award Points Flow – Step-by-Step Implementation

This guide walks through building the **Award Points** Power Automate flow using the Power Automate visual designer.

## 1. Create the flow and trigger

1. In Power Automate, click **Create** → **Automated cloud flow**.
2. Name the flow **Award Points Flow**.
3. Choose trigger **When an item is modified** (SharePoint) and click **Create**.
4. Configure the trigger:
   - **Site Address**: Your SharePoint site URL
   - **List Name**: Onboarding Tasks
5. In the trigger settings (··· → Settings), under **Trigger Conditions**, add the expression:
   ```
   @equals(triggerBody()?['Status'], 'Done')
   ```

## 2. Update task points

1. Add **Get item** (SharePoint):
   - Site Address: same as trigger
   - List Name: Onboarding Tasks
   - Id: **ID** (from trigger)
2. Add **Initialize variable**:
   - Name: `currentPoints`
   - Type: Integer
   - Value: **Points** (from Get item)
3. Add **Compose** (rename to **addPoints**):
   - Inputs: **Points** (from trigger)
4. Add **Compose** (rename to **newPoints**):
   - Inputs (expression):
     ```
     add(variables('currentPoints'), outputs('addPoints'))
     ```
5. Add **Update item** (SharePoint):
   - Site Address, List Name, and Id: same as Get item
   - Points: **Outputs** from newPoints

## 3. (Optional) Notify user in Teams

Add **Post a message (V3)** (Microsoft Teams):
- Team: Your Team
- Channel: Your Channel
- Message:
  ```
  Congrats <at>@{triggerBody()?['Editor']['DisplayName']}</at>! 🎉
  You now have @{outputs('newPoints')} points for completing this task.
  ```
- Enable **Mention user** on the message action.

## 4. Quiz response branch (optional)

To award points for correct quiz answers, use a separate flow (or parallel branch) with these steps:

1. Create an Automated cloud flow with trigger **When a new row is added** (Excel Online (Business)).
2. Configure location, file (`Quizzes.xlsx` in OneDrive), and table (Quizzes).
3. Add **Condition** to compare **Submitted Answer** to **Correct Option**.
4. Under **If yes**, initialize or update points similarly to steps above.

This completes the Award Points flow implementation.
