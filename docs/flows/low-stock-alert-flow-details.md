# Low-Stock Alert Flow – Step-by-Step Implementation

This guide walks through building the **Low-Stock Alert** Power Automate flow to send Teams notifications when inventory items fall below their reorder threshold.

## 1. Create the flow and schedule trigger

1. In Power Automate, click **Create** → **Scheduled cloud flow**.
2. Name the flow **Low-Stock Alert Flow**.
3. Set **Run this flow** every day at 8:00 AM (or as needed) and click **Create**.

## 2. Retrieve inventory data from Excel

1. Add **List rows present in a table** (Excel Online (Business)):
   - Location: OneDrive
   - Document Library: OneDrive
   - File: `InventoryUsage.xlsx`
   - Table: `InventoryUsage`

2. Add **Apply to each**:
   - Select **value** from the previous action.

## 3. Check stock level and send alerts

Inside the **Apply to each** control:

1. Add **Condition**:
   - **Left**: **Stock Level**
   - **Operator**: is less than or equal to
   - **Right**: **Reorder Threshold**

2. Under the **If yes** branch, add **Post adaptive card in a chat or channel** (Microsoft Teams):
   - Post as: **Flow bot**
   - Post in: Your Team and Channel (or choose a 1:1 chat)
   - Adaptive Card JSON:
     ```json
     {
       "type": "AdaptiveCard",
       "body": [
         {
           "type": "TextBlock",
           "weight": "Bolder",
           "size": "Medium",
           "text": "🔔 Low Stock Alert: @{items('Apply_to_each')?['Item Name']}"
         },
         {
           "type": "FactSet",
           "facts": [
             { "title": "Current Stock:", "value": "@{items('Apply_to_each')?['Stock Level']}" },
             { "title": "Threshold:",    "value": "@{items('Apply_to_each')?['Reorder Threshold']}" }
           ]
         }
       ],
       "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
       "version": "1.2"
     }
     ```

3. Save the flow and test it manually or wait for the scheduled run to verify alerts in Teams.
