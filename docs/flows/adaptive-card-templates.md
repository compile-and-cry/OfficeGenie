# Adaptive Card JSON Templates

Copy these Adaptive Card JSON files into Power Automate when using the **Post adaptive card and wait for a response** or **Post adaptive card in a chat or channel** action.

## 1. Quiz Adaptive Card (Multiple Choice)

Save as `templates/quiz-adaptive-card.json`:
```json
{
  "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
  "type": "AdaptiveCard",
  "version": "1.4",
  "body": [
    {
      "type": "TextBlock",
      "text": "📚 Company Onboarding Quiz",
      "weight": "Bolder",
      "size": "Medium"
    },
    {
      "type": "TextBlock",
      "text": "Question: What is our official work timing policy?",
      "wrap": true
    },
    {
      "type": "Input.ChoiceSet",
      "id": "quizAnswer",
      "style": "expanded",
      "choices": [
        { "title": "9–5", "value": "OptionA" },
        { "title": "Flexible 10–4 core hours", "value": "OptionB" },
        { "title": "Shift-based", "value": "OptionC" }
      ]
    }
  ],
  "actions": [
    {
      "type": "Action.Submit",
      "title": "Submit Answer",
      "data": {
        "quizId": "quiz-001",
        "action": "submit_quiz"
      }
    }
  ]
}
```

### Handling Submit Actions for Notifications

After posting the Low Stock Alert card in your flow with **Post adaptive card**:

1. Add a **Switch** action on the value:
   ```
   body('Post_adaptive_card')?['data']['action']
   ```
2. In the **notify_email** case:
   - Add **Send an email (V2)**.
   - Set **To** to your target email address (e.g., `ops-team@yourcompany.com`).
   - Use dynamic content for **Subject** and **Body**, e.g.:
     ```
     Stock alert for @{body('Post_adaptive_card')?['data']['itemId']}
     ```
3. In the **notify_teams** case:
   - Add **Post message in a chat or channel**.
   - Configure **Team** and **Channel** for notifications.
   - Compose the message using dynamic content, e.g.:
     ```
     Inventory alert: @{body('Post_adaptive_card')?['data']['itemId']} is low on stock.
     ```

## 2. Low Stock Alert Adaptive Card

Save as `templates/low-stock-adaptive-card.json`:
```json
{
  "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
  "type": "AdaptiveCard",
  "version": "1.4",
  "body": [
    {
      "type": "TextBlock",
      "text": "⚠️ Low Stock Alert",
      "weight": "Bolder",
      "color": "Attention",
      "size": "Medium"
    },
    { "type": "TextBlock", "text": "**Item:** Printer Paper (A4)", "wrap": true },
    { "type": "TextBlock", "text": "**Current Stock:** 150 sheets", "wrap": true },
    { "type": "TextBlock", "text": "**Reorder Threshold:** 200 sheets", "wrap": true },
    { "type": "TextBlock", "text": "**Suggested Reorder Qty:** 500 sheets", "wrap": true }
  ],
  "actions": [
    {
      "type": "Action.Submit",
      "title": "Notify via Email",
      "data": { "action": "notify_email", "itemId": "item-001" }
    },
    {
      "type": "Action.Submit",
      "title": "Notify in Teams",
      "data": { "action": "notify_teams", "itemId": "item-001" }
    }
  ]
}
```
