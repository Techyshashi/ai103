# MCP Integration and Inventory Agent — Lab Results

## 1. MCP Integration Agent

### Execution

The MCP integration agent was successfully executed using the Python virtual environment.

```powershell
& "g:\My Drive\Azure-Projects\labenv\Scripts\python.exe" "g:/My Drive/Azure-Projects/mslearn-ai-agents/Labfiles/03-mcp-integration/Python/agent.py"
```

### Agent Creation

The agent was successfully created:

```text
Agent created (id: MyAgent:1, name: MyAgent, version: 1)
```

A conversation was also successfully created:

```text
Created conversation (id: conv_...)
```

### MCP Request

The agent was asked:

> Give me the Azure CLI commands to create an Azure Container App with a managed identity.

The agent successfully used the MCP integration to retrieve relevant Microsoft Learn information and returned Azure CLI commands for:

* Azure CLI authentication
* Installing/upgrading the Container Apps extension
* Registering Azure resource providers
* Creating a resource group
* Creating a Container Apps environment
* Creating an Azure Container App
* Assigning a system-assigned managed identity
* Creating and assigning a user-assigned managed identity
* Using managed identities with Azure Container Registry

### Example Commands Returned

```bash
az login

az extension add --name containerapp --upgrade

az provider register --namespace Microsoft.App

az provider register --namespace Microsoft.OperationalInsights

az group create -n $RG -l $LOCATION

az containerapp env create \
    --resource-group $RG \
    --name $ENV \
    --location "$LOCATION"

az containerapp create \
    --resource-group $RG \
    --name $APP \
    --environment $ENV \
    --image $IMAGE
```

System-assigned managed identity:

```bash
az containerapp identity assign \
    --resource-group $RG \
    --name $APP \
    --system-assigned
```

User-assigned managed identity:

```bash
az identity create \
    --resource-group $RG \
    --name <mi-name>

MI_ID=$(az identity show \
    --resource-group $RG \
    --name <mi-name> \
    --query id \
    --output tsv)

az containerapp identity assign \
    --resource-group $RG \
    --name $APP \
    --user-assigned $MI_ID
```

### Result

```text
Agent deleted
```

The MCP integration test completed successfully.

---

# 2. Inventory Agent

## Client Execution

The inventory agent was successfully connected to the MCP server.

Available tools:

```text
['get_inventory_levels', 'get_weekly_sales']
```

The agent provided an interactive prompt:

```text
Enter a prompt for the inventory agent. Use 'quit' to exit.
```

---

## Test 1 — Current Inventory

### User Prompt

```text
Show me the current inventory levels for all products.
```

### Agent Response

| Product        | Inventory |
| -------------- | --------: |
| Moisturizer    |         6 |
| Shampoo        |         8 |
| Body Spray     |        28 |
| Hair Gel       |         5 |
| Lip Balm       |        12 |
| Skin Serum     |         9 |
| Cleanser       |        30 |
| Conditioner    |         3 |
| Setting Powder |        17 |
| Dry Shampoo    |        45 |

The agent successfully retrieved the current inventory levels using the inventory tool.

---

# 3. Test 2 — Products Requiring Restock

### User Prompt

```text
Are there any products that should be restocked?
```

### Agent Logic

The agent used the following criteria:

```text
Inventory < 10
AND
Weekly sales > 15
```

### Agent Response

| Product     | Inventory | Weekly Sales | Recommendation |
| ----------- | --------: | -----------: | -------------- |
| Moisturizer |         6 |           22 | Restock        |
| Shampoo     |         8 |           18 | Restock        |
| Skin Serum  |         9 |           19 | Restock        |

### Result

Three products were identified as requiring restocking:

1. **Moisturizer** — inventory: 6, weekly sales: 22
2. **Shampoo** — inventory: 8, weekly sales: 18
3. **Skin Serum** — inventory: 9, weekly sales: 19

---

# 4. Test 3 — Clearance Recommendation

### User Prompt

```text
Which products would you recommend for clearance?
```

### Agent Response

The agent returned:

```text
Recommended for restock:
- Moisturizer — inventory 6, weekly sales 22
- Shampoo — inventory 8, weekly sales 18
- Skin Serum — inventory 9, weekly sales 19
```

### Observation

The response appears inconsistent with the user's request.

The agent returned the same products recommended for **restocking** rather than identifying products for **clearance**.

This indicates that the agent's decision-making instructions or tool logic should be reviewed.

A possible clearance rule could be based on:

```text
High inventory
AND
Low weekly sales
```

For example:

```text
Inventory > threshold
AND
Weekly sales < threshold
→ Clearance
```

The exact thresholds should be defined by the business requirements.

---

# 5. Tool Integration

The inventory agent successfully connected to two tools:

```text
get_inventory_levels
get_weekly_sales
```

The workflow can be represented as:

```text
User
  │
  ▼
Inventory Agent
  │
  ├── get_inventory_levels
  │
  └── get_weekly_sales
          │
          ▼
     Business Rules
          │
          ├── Restock
          │
          └── Clearance
          │
          ▼
     Agent Response
```

---

# 6. Overall Test Result

| Test                     | Result                   |
| ------------------------ | ------------------------ |
| MCP server connection    | ✅ Successful             |
| MCP tools discovered     | ✅ Successful             |
| Inventory retrieval      | ✅ Successful             |
| Weekly sales retrieval   | ✅ Successful             |
| Restock recommendation   | ✅ Successful             |
| Clearance recommendation | ⚠️ Logic requires review |
| Interactive conversation | ✅ Successful             |
| Agent cleanup            | ✅ Successful             |

---

# 7. Agent Cleanup

After testing, the user entered:

```text
quit
```

The application exited the interactive chat and cleaned up the agent:

```text
Exiting chat.
Cleaning up agents:
Deleted inventory agent.
```

This confirms that the agent lifecycle was handled correctly.

---

# 8. Key Learning

This lab demonstrated how an AI agent can interact with external tools through MCP and use information retrieved from those tools to make decisions.

The overall architecture is:

```text
┌──────────────┐
│     User     │
└──────┬───────┘
       │
       ▼
┌──────────────────┐
│   AI Agent       │
│                  │
│ Reasoning +      │
│ Tool Selection   │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│   MCP Server     │
└────────┬─────────┘
         │
         ├─────────────────┐
         ▼                 ▼
┌────────────────┐  ┌────────────────┐
│ Inventory Tool │  │ Sales Tool     │
│                │  │                │
│ get_inventory  │  │ get_weekly_    │
│ _levels        │  │ sales          │
└───────┬────────┘  └───────┬────────┘
        │                   │
        └─────────┬─────────┘
                  ▼
          ┌───────────────┐
          │ Business      │
          │ Rules         │
          └───────┬───────┘
                  │
             ┌────┴────┐
             ▼         ▼
        ┌─────────┐ ┌───────────┐
        │ Restock │ │ Clearance │
        └────┬────┘ └─────┬─────┘
             │            │
             └─────┬──────┘
                   ▼
            ┌────────────┐
            │   Agent    │
            │  Response  │
            └────────────┘
```

## Final Status

**MCP Integration:** ✅ Completed successfully

**Inventory Agent:** ✅ Successfully executed

**Tool Integration:** ✅ Working

**Restock Analysis:** ✅ Working

**Clearance Analysis:** ⚠️ Requires improvement to the agent's business logic

**Agent Cleanup:** ✅ Successful
