# GTM API Request Templates for LLMs

Complete, copy-paste request bodies for common operations.

---

## Tag Templates

### GA4 Configuration Tag

```json
{
  "name": "GA4 Configuration",
  "type": "gaawe",
  "firingTriggerId": ["TRIGGER_ID"],
  "parameter": [
    {
      "type": "template",
      "key": "measurementId",
      "value": "G-XXXXXXXXXX"
    },
    {
      "type": "list",
      "key": "configParameter",
      "list": [
        {
          "type": "map",
          "map": [
            {
              "type": "template",
              "key": "name",
              "value": "send_page_view"
            },
            {
              "type": "template",
              "key": "value",
              "value": "false"
            }
          ]
        }
      ]
    }
  ]
}
```

**Variables to replace:**
- `TRIGGER_ID`: ID of trigger (e.g., "5")
- `G-XXXXXXXXXX`: Your GA4 Measurement ID

---

### GA4 Event Tag

```json
{
  "name": "GA4 Event - Custom Event",
  "type": "gaawe",
  "firingTriggerId": ["TRIGGER_ID"],
  "parameter": [
    {
      "type": "template",
      "key": "measurementId",
      "value": "G-XXXXXXXXXX"
    },
    {
      "type": "template",
      "key": "eventName",
      "value": "custom_event_name"
    },
    {
      "type": "list",
      "key": "eventParameters",
      "list": [
        {
          "type": "map",
          "map": [
            {
              "type": "template",
              "key": "name",
              "value": "parameter_name"
            },
            {
              "type": "template",
              "key": "value",
              "value": "{{Variable Name}}"
            }
          ]
        }
      ]
    }
  ]
}
```

**Variables to replace:**
- `TRIGGER_ID`: Trigger ID
- `G-XXXXXXXXXX`: GA4 Measurement ID
- `custom_event_name`: Event name
- `parameter_name`: Event parameter name
- `{{Variable Name}}`: Variable reference

---

### Custom HTML Tag

```json
{
  "name": "Custom HTML - Tracking Script",
  "type": "html",
  "firingTriggerId": ["TRIGGER_ID"],
  "parameter": [
    {
      "type": "template",
      "key": "html",
      "value": "<script>\n  // Your custom JavaScript here\n  console.log('Custom tracking');\n</script>"
    },
    {
      "type": "boolean",
      "key": "supportDocumentWrite",
      "value": "false"
    }
  ]
}
```

**Variables to replace:**
- `TRIGGER_ID`: Trigger ID
- HTML content in `value` field

---

### Custom Image Tag

```json
{
  "name": "Custom Image - Pixel",
  "type": "img",
  "firingTriggerId": ["TRIGGER_ID"],
  "parameter": [
    {
      "type": "template",
      "key": "url",
      "value": "https://example.com/pixel.gif?id={{Page URL}}"
    },
    {
      "type": "boolean",
      "key": "cacheBusterQueryParam",
      "value": "true"
    }
  ]
}
```

**Variables to replace:**
- `TRIGGER_ID`: Trigger ID
- URL in `value` field

---

### Google Ads Conversion Tag

```json
{
  "name": "Google Ads Conversion",
  "type": "awct",
  "firingTriggerId": ["TRIGGER_ID"],
  "parameter": [
    {
      "type": "template",
      "key": "conversionId",
      "value": "AW-123456789"
    },
    {
      "type": "template",
      "key": "conversionLabel",
      "value": "AbCdEfGhIjKlMnOp"
    },
    {
      "type": "template",
      "key": "conversionValue",
      "value": "{{Transaction Total}}"
    },
    {
      "type": "template",
      "key": "conversionCurrency",
      "value": "USD"
    },
    {
      "type": "template",
      "key": "orderId",
      "value": "{{Transaction ID}}"
    }
  ]
}
```

**Variables to replace:**
- `TRIGGER_ID`: Trigger ID
- `AW-123456789`: Conversion ID
- `AbCdEfGhIjKlMnOp`: Conversion label
- Variable references as needed

---

### Google Ads Conversion Linker

```json
{
  "name": "Google Ads Conversion Linker",
  "type": "gclidw",
  "firingTriggerId": ["TRIGGER_ID"],
  "parameter": []
}
```

**Variables to replace:**
- `TRIGGER_ID`: Usually "All Pages" trigger

---

### Tag with Blocking Trigger

```json
{
  "name": "Tag with Exception",
  "type": "html",
  "firingTriggerId": ["TRIGGER_ID_FIRE"],
  "blockingTriggerId": ["TRIGGER_ID_BLOCK"],
  "parameter": [
    {
      "type": "template",
      "key": "html",
      "value": "<script>console.log('Fires except on blocked pages');</script>"
    },
    {
      "type": "boolean",
      "key": "supportDocumentWrite",
      "value": "false"
    }
  ]
}
```

**Variables to replace:**
- `TRIGGER_ID_FIRE`: Firing trigger ID
- `TRIGGER_ID_BLOCK`: Blocking trigger ID (exception)

---

### Tag with Scheduling

```json
{
  "name": "Scheduled Campaign Tag",
  "type": "html",
  "firingTriggerId": ["TRIGGER_ID"],
  "scheduleStartMs": "1704067200000",
  "scheduleEndMs": "1735689600000",
  "parameter": [
    {
      "type": "template",
      "key": "html",
      "value": "<script>console.log('Campaign active');</script>"
    },
    {
      "type": "boolean",
      "key": "supportDocumentWrite",
      "value": "false"
    }
  ]
}
```

**Variables to replace:**
- `TRIGGER_ID`: Trigger ID
- `scheduleStartMs`: Start timestamp in milliseconds
- `scheduleEndMs`: End timestamp in milliseconds

---

## Trigger Templates

### All Pages Trigger

```json
{
  "name": "All Pages",
  "type": "pageview"
}
```

No variables to replace - this is complete.

---

### Custom Event Trigger

```json
{
  "name": "Custom Event - Purchase",
  "type": "customEvent",
  "customEventFilter": [
    {
      "type": "equals",
      "parameter": [
        {
          "type": "template",
          "key": "arg0",
          "value": "{{_event}}"
        },
        {
          "type": "template",
          "key": "arg1",
          "value": "purchase"
        }
      ]
    }
  ]
}
```

**Variables to replace:**
- `purchase`: Your custom event name (the dataLayer event name to match)

**Note:** Use `customEventFilter` (not `eventName`) for customEvent triggers. The `{{_event}}` variable references the dataLayer event name.

---

### Page View with Filter

```json
{
  "name": "Page View - Thank You Page",
  "type": "pageview",
  "filter": [
    {
      "type": "contains",
      "parameter": [
        {
          "type": "template",
          "key": "arg0",
          "value": "{{Page Path}}"
        },
        {
          "type": "template",
          "key": "arg1",
          "value": "/thank-you"
        }
      ]
    }
  ]
}
```

**Variables to replace:**
- `{{Page Path}}`: Variable reference (usually built-in)
- `/thank-you`: Path to match

---

### Click Trigger

```json
{
  "name": "Click - CTA Button",
  "type": "linkClick",
  "autoEventFilter": [
    {
      "type": "equals",
      "parameter": [
        {
          "type": "template",
          "key": "arg0",
          "value": "{{Click ID}}"
        },
        {
          "type": "template",
          "key": "arg1",
          "value": "cta-button"
        }
      ]
    }
  ],
  "waitForTags": {
    "type": "boolean",
    "value": "true"
  },
  "waitForTagsTimeout": {
    "type": "integer",
    "value": "2000"
  }
}
```

**Variables to replace:**
- `cta-button`: Element ID to match

---

### Form Submission Trigger

```json
{
  "name": "Form Submission - Contact Form",
  "type": "formSubmission",
  "autoEventFilter": [
    {
      "type": "equals",
      "parameter": [
        {
          "type": "template",
          "key": "arg0",
          "value": "{{Form ID}}"
        },
        {
          "type": "template",
          "key": "arg1",
          "value": "contact-form"
        }
      ]
    }
  ],
  "waitForTags": {
    "type": "boolean",
    "value": "true"
  },
  "checkValidation": {
    "type": "boolean",
    "value": "true"
  }
}
```

**Variables to replace:**
- `contact-form`: Form ID to match

---

### Timer Trigger

```json
{
  "name": "Timer - Every 30 Seconds",
  "type": "timer",
  "interval": {
    "type": "integer",
    "value": "30000"
  },
  "limit": {
    "type": "integer",
    "value": "10"
  }
}
```

**Variables to replace:**
- `30000`: Interval in milliseconds
- `10`: Maximum number of fires (optional)

---

### Scroll Depth Trigger

```json
{
  "name": "Scroll Depth - 75%",
  "type": "scrollDepth",
  "filter": [
    {
      "type": "equals",
      "parameter": [
        {
          "type": "template",
          "key": "arg0",
          "value": "{{Scroll Depth Threshold}}"
        },
        {
          "type": "template",
          "key": "arg1",
          "value": "75"
        }
      ]
    }
  ]
}
```

**Variables to replace:**
- `75`: Scroll percentage threshold

---

### Element Visibility Trigger

```json
{
  "name": "Element Visibility - Hero Banner",
  "type": "elementVisibility",
  "filter": [
    {
      "type": "css",
      "parameter": [
        {
          "type": "template",
          "key": "arg0",
          "value": "#hero-banner"
        }
      ]
    }
  ]
}
```

**Variables to replace:**
- `#hero-banner`: CSS selector

---

### Trigger with Multiple Filters (AND)

```json
{
  "name": "Checkout - Logged In Users",
  "type": "pageview",
  "filter": [
    {
      "type": "contains",
      "parameter": [
        {
          "type": "template",
          "key": "arg0",
          "value": "{{Page Path}}"
        },
        {
          "type": "template",
          "key": "arg1",
          "value": "/checkout"
        }
      ]
    },
    {
      "type": "equals",
      "parameter": [
        {
          "type": "template",
          "key": "arg0",
          "value": "{{User Status}}"
        },
        {
          "type": "template",
          "key": "arg1",
          "value": "logged_in"
        }
      ]
    }
  ]
}
```

**Note:** Multiple filters in array = AND logic. All must match.

---

## Variable Templates

### Constant Variable

```json
{
  "name": "GA4 Measurement ID",
  "type": "c",
  "parameter": [
    {
      "type": "template",
      "key": "value",
      "value": "G-XXXXXXXXXX"
    }
  ]
}
```

**Variables to replace:**
- Variable name
- Constant value

---

### Data Layer Variable

```json
{
  "name": "Transaction ID",
  "type": "v",
  "parameter": [
    {
      "type": "template",
      "key": "name",
      "value": "ecommerce.transaction_id"
    },
    {
      "type": "integer",
      "key": "dataLayerVersion",
      "value": "2"
    }
  ]
}
```

**Variables to replace:**
- Variable name
- `ecommerce.transaction_id`: Data layer key path

---

### Data Layer Variable with Default

```json
{
  "name": "User Type",
  "type": "v",
  "parameter": [
    {
      "type": "template",
      "key": "name",
      "value": "user.type"
    },
    {
      "type": "template",
      "key": "defaultValue",
      "value": "guest"
    },
    {
      "type": "integer",
      "key": "dataLayerVersion",
      "value": "2"
    }
  ]
}
```

**Variables to replace:**
- Data layer key
- Default value

---

### First-Party Cookie Variable

```json
{
  "name": "Session ID Cookie",
  "type": "k",
  "parameter": [
    {
      "type": "template",
      "key": "name",
      "value": "session_id"
    },
    {
      "type": "boolean",
      "key": "decodeCookie",
      "value": "true"
    }
  ]
}
```

**Variables to replace:**
- Cookie name

---

### Custom JavaScript Variable

```json
{
  "name": "Current Timestamp",
  "type": "jsm",
  "parameter": [
    {
      "type": "template",
      "key": "javascript",
      "value": "function() {\n  return Date.now();\n}"
    }
  ]
}
```

**Variables to replace:**
- JavaScript function code

---

### URL Variable

```json
{
  "name": "Query Parameter - utm_source",
  "type": "u",
  "parameter": [
    {
      "type": "template",
      "key": "component",
      "value": "QUERY"
    },
    {
      "type": "template",
      "key": "queryKey",
      "value": "utm_source"
    }
  ]
}
```

**Component values:**
- `URL`: Full URL
- `HOST`: Hostname
- `PATH`: Path
- `QUERY`: Query string
- `FRAGMENT`: URL fragment
- `PORT`: Port number
- `PROTOCOL`: Protocol (http/https)

---

### Variable with Format

```json
{
  "name": "Lowercase Page Path",
  "type": "v",
  "parameter": [
    {
      "type": "template",
      "key": "name",
      "value": "page.path"
    }
  ],
  "formatValue": {
    "caseConversionType": "lowercase"
  }
}
```

**Case conversion options:**
- `lowercase`
- `uppercase`
- `none`

---

### Variable with Null Conversion

```json
{
  "name": "User ID with Default",
  "type": "v",
  "parameter": [
    {
      "type": "template",
      "key": "name",
      "value": "user.id"
    }
  ],
  "formatValue": {
    "convertNullToValue": {
      "type": "template",
      "value": "anonymous"
    },
    "convertUndefinedToValue": {
      "type": "template",
      "value": "anonymous"
    }
  }
}
```

---

## Workspace Templates

### Create Workspace

```json
{
  "name": "Add New Feature",
  "description": "Workspace for implementing new tracking feature"
}
```

---

### Create Version from Workspace

```json
{
  "name": "v1.2.0 - Add GA4 tracking",
  "notes": "- Added GA4 configuration tag\n- Added GA4 event tags for key conversions\n- Updated all pages trigger"
}
```

---

## Environment Templates

### Create Environment

```json
{
  "name": "Staging",
  "description": "Staging environment for testing",
  "type": "user"
}
```

**Environment types:**
- `user`: Custom environment
- `live`: Production (usually auto-created)
- `latest`: Latest version (usually auto-created)

---

## Complete Request Examples

### Create Tag Request

```http
POST https://tagmanager.googleapis.com/tagmanager/v2/accounts/123/containers/456/workspaces/10/tags
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "name": "GA4 - Page View",
  "type": "gaawe",
  "firingTriggerId": ["5"],
  "parameter": [
    {
      "type": "template",
      "key": "measurementId",
      "value": "G-ABC123XYZ"
    }
  ]
}
```

---

### Update Tag Request

```http
PUT https://tagmanager.googleapis.com/tagmanager/v2/accounts/123/containers/456/workspaces/10/tags/15
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "path": "accounts/123/containers/456/workspaces/10/tags/15",
  "tagId": "15",
  "name": "GA4 - Page View + Scroll",
  "type": "gaawe",
  "firingTriggerId": ["5", "7"],
  "fingerprint": "1234567890",
  "parameter": [
    {
      "type": "template",
      "key": "measurementId",
      "value": "G-ABC123XYZ"
    }
  ]
}
```

**Note:** Include ALL fields from GET response, not just changed fields.

---

### Create Version Request

```http
POST https://tagmanager.googleapis.com/tagmanager/v2/accounts/123/containers/456/workspaces/10:create_version
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "name": "Version 5",
  "notes": "Added new tracking tags"
}
```

---

### Publish Version Request

```http
POST https://tagmanager.googleapis.com/tagmanager/v2/accounts/123/containers/456/versions/5:publish
Authorization: Bearer {access_token}
Content-Type: application/json

{}
```

**Note:** Body can be empty.

---

## Template Usage Guide for LLMs

### Step 1: Choose template
Identify the entity type you need to create.

### Step 2: Copy template
Copy the complete JSON template.

### Step 3: Replace variables
Replace ALL placeholders marked with:
- `TRIGGER_ID` → actual trigger ID
- `G-XXXXXXXXXX` → actual measurement ID
- `{{Variable Name}}` → actual variable reference
- etc.

### Step 4: Validate
Check against validation rules:
- All required fields present
- No auto-generated fields included (for create)
- Parameter structure is valid
- References exist in workspace

### Step 5: Make request
Send to appropriate endpoint with proper auth.

---

## Common Mistakes to Avoid

```
❌ WRONG: Including auto-generated fields in create request
{
  "tagId": "15",  // ← Remove this
  "path": "...",  // ← Remove this
  "name": "My Tag"
}

✅ CORRECT: Only user-provided fields
{
  "name": "My Tag",
  "type": "html",
  "firingTriggerId": ["5"]
}
```

```
❌ WRONG: Missing fingerprint in update
PUT /tags/15
{
  "name": "Updated Name"
}

✅ CORRECT: Include full entity with fingerprint
PUT /tags/15
{
  "path": "accounts/123/.../tags/15",
  "tagId": "15",
  "name": "Updated Name",
  "type": "html",
  "firingTriggerId": ["5"],
  "fingerprint": "current_fingerprint",
  "parameter": [...]
}
```

```
❌ WRONG: Boolean as actual boolean
{
  "type": "boolean",
  "key": "paused",
  "value": true  // ← Should be string
}

✅ CORRECT: Boolean as string
{
  "type": "boolean",
  "key": "paused",
  "value": "true"  // ← String "true" or "false"
}
```

```
❌ WRONG: Integer as number
{
  "type": "integer",
  "key": "interval",
  "value": 30000  // ← Should be string
}

✅ CORRECT: Integer as string
{
  "type": "integer",
  "key": "interval",
  "value": "30000"  // ← String representation
}
```
