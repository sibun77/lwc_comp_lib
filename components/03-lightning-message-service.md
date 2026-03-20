← [Back to Index](../README.md)

---

## 3. Lightning Message Service Communication

| Property | Value |
|--------|--------|
| **Component Name** | `pubLWC`, `subLWC` |
| **Category** | Component Communication |
| **Type** | Utility / Advanced Communication |
| **Description** | Demonstrates communication between unrelated components using Lightning Message Service (LMS). This allows components to exchange data without a direct parent-child relationship. |

---

### Use Cases

- **Unrelated Component Communication** — Connect components with no shared DOM hierarchy.
- **Cross DOM Messaging** — Send messages across different parts of the page.
- **App-wide Event Handling** — Broadcast global notifications.
- **Decoupled Component Architecture** — Keep components independent while still reactive.

---

### Implementation

#### Publisher — `pubLWC.html`

```html
<template>
    <lightning-card title="Publisher Component" variant="base">
        <div class="slds-m-around_small">
            <lightning-input
                type="text"
                label="Enter Message"
                value={value}
                onchange={HandleChange}>
            </lightning-input>
            <button
                class="slds-button slds-button_success slds-m-top_small"
                onclick={handleMessageSend}>
                Send Message
            </button>
        </div>
    </lightning-card>
</template>
```

#### Publisher — `pubLWC.js`

```javascript
import { LightningElement, track, wire } from 'lwc';
import { publish, MessageContext } from 'lightning/messageService';
import COUNTING_UPDATED_CHANNEL from '@salesforce/messageChannel/Counting_Update__c';

export default class PubLWC extends LightningElement {
    @wire(MessageContext) messageContext;
    @track value;

    HandleChange(event) {
        this.value = event.target.value;
    }
    handleMessageSend() {
        const payload = {
            field: 'msg',
            constant: this.value
        };
        publish(this.messageContext, COUNTING_UPDATED_CHANNEL, payload);
    }
}
```

#### Subscriber — `subLWC.html`

```html
<template>
    <lightning-card title="Subscriber Component" variant="base">
        <p class="slds-m-around_small">
            Message: {msg}
        </p>
    </lightning-card>
</template>
```

#### Subscriber — `subLWC.js`

```javascript
import { LightningElement, wire } from 'lwc';
import { subscribe, MessageContext } from 'lightning/messageService';
import COUNTING_UPDATED_CHANNEL from '@salesforce/messageChannel/Counting_Update__c';

export default class SubLWC extends LightningElement {
    subscription = null;
    @wire(MessageContext) messageContext;
    msg;

    connectedCallback() {
        this.subscribeToMessageChannel();
    }
    subscribeToMessageChannel() {
        this.subscription = subscribe(
            this.messageContext,
            COUNTING_UPDATED_CHANNEL,
            (message) => this.handleMessage(message)
        );
    }
    handleMessage(message) {
        if (message.field === 'msg') {
            this.msg = message.constant;
        }
    }
}
```

#### Message Channel — `Counting_Update__c.messageChannel-meta.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<LightningMessageChannel xmlns="http://soap.sforce.com/2006/04/metadata">
    <masterLabel>Component Communication Channel</masterLabel>
    <isExposed>true</isExposed>
    <description>Channel for communication between ComponentA and ComponentB</description>
</LightningMessageChannel>
```

---

### Preview

![Lightning Message Service LWC](./images/lms_component_lwc.png)
