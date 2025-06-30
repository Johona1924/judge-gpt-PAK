# Using Two Azure OpenAI Models in JudgeGPT

This guide explains how to configure and use two Azure OpenAI models in JudgeGPT. The application supports conditional routing between two models based on user IDs.

---

## **Overview**

JudgeGPT allows you to configure two Azure OpenAI models:
1. **Primary Model**: The default model used for most users
2. **Alternative Model**: A secondary model used for users with a userID from a provided selection

The application dynamically initializes and routes requests to the appropriate model based on the configuration in the `.env` file and the logic in the `app.py` file. Additionally, the title for conversations is always computed using the primary model, regardless of user routing.

---

## **Configuration Steps**

### 1. **Environment Variables**
The `.env` file contains the configuration for both models. Below are the key variables:

```properties
# Primary Model Configuration
AZURE_OPENAI_MODEL=gpt-4o-mini
AZURE_OPENAI_KEY=<primary_model_api_key>
AZURE_OPENAI_RESOURCE=<primary_model_resource_name>
AZURE_OPENAI_ENDPOINT=https://<primary_model_resource_name>.openai.azure.com/

# Alternative Model Configuration (optional)
AZURE_OPENAI_ALT_MODEL=gpt-4-turbo
AZURE_OPENAI_KEY_2=<alt_model_api_key>
AZURE_OPENAI_RESOURCE_2=<alt_model_resource_name>
AZURE_OPENAI_ENDPOINT_2=https://<alt_model_resource_name>.openai.azure.com/
AZURE_OPENAI_PREVIEW_API_VERSION_2=2023-12-01-preview

# User Routing for Alternative Model
AZURE_OPENAI_ALT_MODEL_USER_IDS=00000000-0000-0000-0000-000000000000,auth0|6855852b951d58e1da2c2179
```

#### **Primary Model Settings**:
- `AZURE_OPENAI_MODEL`: Name of the primary model deployment (required)
- `AZURE_OPENAI_KEY`: API key for the primary model (optional if using managed identity)
- `AZURE_OPENAI_RESOURCE`: Azure resource name for the primary model (required if AZURE_OPENAI_ENDPOINT not set)
- `AZURE_OPENAI_ENDPOINT`: Full endpoint URL for the primary model (required if AZURE_OPENAI_RESOURCE not set)

#### **Alternative Model Settings**:
- `AZURE_OPENAI_ALT_MODEL`: Name of the alternative model deployment (optional)
- `AZURE_OPENAI_KEY_2`: API key for the alternative model (optional if using managed identity)
- `AZURE_OPENAI_RESOURCE_2`: Azure resource name for the alternative model (required if AZURE_OPENAI_ENDPOINT_2 not set)
- `AZURE_OPENAI_ENDPOINT_2`: Full endpoint URL for the alternative model (required if AZURE_OPENAI_RESOURCE_2 not set)
- `AZURE_OPENAI_PREVIEW_API_VERSION_2`: API version for the alternative model (defaults to 2023-12-01-preview)

#### **User Routing Settings**:
- `AZURE_OPENAI_ALT_MODEL_USER_IDS`: Comma-separated string of user IDs that should use the alternative model. **Do NOT use double-quotes when setting the value in .env or in Azure App Service Environment Variables**

### 2. **Single vs. Dual Model Usage**

#### **Single Model (Default)**:
- To use only the primary model, leave `AZURE_OPENAI_ALT_MODEL` empty or unset in the `.env` file
- All users will be routed to the primary model

#### **Dual Model**:
- To enable both models, set `AZURE_OPENAI_ALT_MODEL` and its related variables in the `.env` file
- Users listed in `AZURE_OPENAI_ALT_MODEL_USER_IDS` will be routed to the alternative model
- All other users will be routed to the primary model

### 3. **User ID Based Routing**

The backend uses the `user_principal_id` from the authenticated user's details to determine which model to route the request to:

- If the `user_principal_id` is in the `AZURE_OPENAI_ALT_MODEL_USER_IDS` list, the request is routed to the alternative model
- Otherwise, it is routed to the primary model
- **Title generation always uses the primary model**, regardless of user routing

---

## **Example Configuration**

### **Single Model Setup**
```properties
AZURE_OPENAI_MODEL=gpt-4o-mini
AZURE_OPENAI_RESOURCE=my-openai-resource
AZURE_OPENAI_KEY=your-api-key-here
```

### **Dual Model Setup**
```properties
# Primary Model
AZURE_OPENAI_MODEL=gpt-4o-mini
AZURE_OPENAI_RESOURCE=my-openai-resource
AZURE_OPENAI_KEY=your-primary-api-key-here

# Alternative Model
AZURE_OPENAI_ALT_MODEL=gpt-4.1-mini
AZURE_OPENAI_RESOURCE_2=my-premium-openai-resource
AZURE_OPENAI_KEY_2=your-alt-api-key-here

# Users who should use the alternative model
AZURE_OPENAI_ALT_MODEL_USER_IDS=auth0|abc123def456,00000000-0000-0000-0000-000000000001
```

