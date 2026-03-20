← [Back to Index](../README.md)

---

## 2. Child → Parent Communication (Custom Event)

| Property | Value |
|--------|--------|
| **Component Name** | `customEventParent` |
| **Category** | Component Communication |
| **Type** | Utility / Learning Component |
| **Description** | Demonstrates how a child component can send data back to its parent using a CustomEvent. This pattern is widely used in Lightning Web Components for upward data flow. |

---

### Use Cases

- **Child → Parent Data Communication** — Send input values up to the parent.
- **Form Submission Handling** — Trigger parent-level logic from child actions.
- **Reusable Input Components** — Notify parent components of changes.
- **Dynamic UI Updates** — Update parent state based on child events.

---

### Implementation

#### Parent — `customEventParent.html`

```html
<template>
    <lightning-card>
        <div class="slds-grid slds-m-around_medium">
            <div class="slds-col slds-m-around_small">
                <lightning-card title="Parent Component">
                    <div class="slds-m-left_small">
                        Hello, {name}!
                    </div>
                </lightning-card>
            </div>
            <div class="slds-col slds-m-around_small">
                <c-custom-event-child onchildevt={handleChildEvent}></c-custom-event-child>
            </div>
        </div>
    </lightning-card>
</template>
```

#### Parent — `customEventParent.js`

```javascript
import { LightningElement } from 'lwc';

export default class CustomEventParent extends LightningElement {
    name = '';

    handleChildEvent(event) {
        this.name = event.detail;
    }
}
```

#### Child — `customEventChild.html`

```html
<template>
    <lightning-card title="Child Component">
        <div class="slds-col slds-m-around_small">
            <lightning-input type="text" label="Enter Name" value={inputText} onchange={handleChange}>
            </lightning-input>
            <button class="slds-button slds-button_brand slds-m-top_small" onclick={sendToParent}>
                Submit
            </button>
        </div>
    </lightning-card>
</template>
```

#### Child — `customEventChild.js`

```javascript
import { LightningElement } from 'lwc';

export default class CustomEventChild extends LightningElement {
    name = '';

    handleChange(event) {
        this.name = event.target.value;
    }
    sendToParent() {
        let evt = new CustomEvent('childevt', {
            detail: this.name
        });
        this.dispatchEvent(evt);
    }
}
```

---

### Preview

![Custom Event Communication LWC](./images/custom_event_lwc.png)
