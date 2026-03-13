# 🧩 LWC Utility Components Library

A personal collection of reusable **Lightning Web Components (LWC)** that can be quickly integrated into Salesforce projects without rewriting code from scratch.

This documentation will grow over time as more reusable components are added.

---

# 📚 Component Index

| # | Component | Description |
|---|---|---|
| 1 | [WebCam Image Capture Component](#1️⃣-lwc-webcam-image-capture-component-) | Capture images directly from the user's webcam using MediaDevices API |
| 2 | [Dependent Picklist Component](#2️⃣-lwc-dependent-picklist-component-) | Dynamic Country → State → City dependent picklist |

---

# 1️⃣ LWC WebCam Image Capture Component 📸

| Property | Value |
|--------|--------|
| **Component Name** | `webCamImageLwc` |
| **Category** | Media / Camera |
| **Type** | Utility Component |
| **Description** | A reusable Lightning Web Component that interfaces with the browser MediaDevices API to stream video and capture still images directly inside Salesforce UI. |

---

## 🚀 Use Cases

This component can be used in multiple real-world Salesforce scenarios:

- **Field Service**  
  Capture equipment or site photos.

- **Identity Verification**  
  Quickly capture a user photo for validation.

- **Case Documentation**  
  Attach real-time images to a Case or Lead.

- **Inspection Systems**  
  Capture product condition images.

---

## 🛠 Implementation

### HTML Template  
`webCamImageLwc.html`

```html
<template>
    <lightning-card>
        <div>
            <lightning-button label="Start Video" onclick={startCamera}></lightning-button>
            <lightning-button label="Stop Video" onclick={stopCamera}></lightning-button>
            <lightning-button label="Capture Photo" onclick={captureImage}></lightning-button>
        </div>
        <div class="slds-grid slds-p-around_medium">
            <div class="slds-col">
                <video class="videoElement" width="320" height="240" autoplay></video>
            </div>
            <div class="slds-col">
                <img src="" class="slds-hide imageElement" alt="Captured Image" width="320" height="240"/>
            </div>
            <canvas class="slds-hide canvas"></canvas>
        </div>
    </lightning-card>
</template>
```
### JavaScript Template  
`webCamImageLwc.js`

```Javascript
import { LightningElement } from 'lwc';

export default class WebCamImageLwc extends LightningElement {
    videoElement;
    imageElement;
    canvasElement;
    renderedCallback() {
        this.videoElement = this.template.querySelector('.videoElement');
        this.canvasElement = this.template.querySelector('.canvas');
    }
    async startCamera() {
        if(navigator.mediaDevices && navigator.mediaDevices.getUserMedia){
            try {
                this.videoElement.srcObject = await navigator.mediaDevices.getUserMedia({video:true,audio:false});
            } catch (error) {
                console.log('error', error);
            }
        }else{
            console.log('Get user Media is not supported');
        }
    }
    async stopCamera() {
        const video = this.template.querySelector('.videoElement');
        video.srcObject.getTracks().forEach( (track) => track.stop());
        video.srcObject = null;
        this.hideImageElement();
    }
    captureImage() {
        if(this.videoElement && this.videoElement.srcObject!=null){
            this.canvasElement.height = this.videoElement.videoHeight;
            this.canvasElement.width = this.videoElement.videoWidth;
            const context = this.canvasElement.getContext('2d');
            context.drawImage(this.videoElement,0,0,this.canvasElement.width,this.canvasElement.height);
            const imgData = this.canvasElement.toDataURL('image/png');
            const imageElement = this.template.querySelector('.imageElement');
            imageElement.setAttribute('src',imgData);
            imageElement.classList.add('slds-show');
            imageElement.classList.remove('slds-hide');
        }
    }
    hideImageElement(){
        const imageElement = this.template.querySelector('.imageElement');
        imageElement.classList.add('slds-hide');
        imageElement.classList.remove('slds-show');
    }
}
```

### 📷 Component Preview
![Image Capturing Lwc Component](./images/image_capture_lwc.png)

# 2️⃣ LWC Dependent Picklist Component 🌍

| Property | Value |
|--------|--------|
| **Component Name** | `dependantPicklistComp` |
| **Category** | Form / Data Selection |
| **Type** | Utility Component |
| **Description** | A reusable Lightning Web Component that implements a three-level dependent picklist (Country → State → City) using Apex data. The component dynamically enables and disables fields based on the user's selection. |

---

# 🚀 Use Cases

This component can be used in multiple Salesforce scenarios:

- **Address Forms**  
  Dynamically select Country → State → City.

- **Customer Registration Systems**  
  Capture hierarchical location information.

- **Lead / Account Creation**  
  Improve data accuracy with dependent selections.

- **Survey or Data Collection Apps**  
  Guide users through structured input.

---

# 🛠 Implementation

## HTML Template  
`dependantPicklistComp.html`

```html
<template>
    <lightning-card title="Dependant Picklist" icon-name="custom:custom14">
        <div class="slds-m-around_medium">
            <lightning-combobox label="Select Country" value={country} options={countryOptions}
                onchange={handleCountryChange}>
            </lightning-combobox>
            <lightning-combobox label="Select State" value={state} options={stateOptions} onchange={handleStateChange}
                disabled={isStateDisabled}>
            </lightning-combobox>
            <lightning-combobox label="Select City" value={city} options={cityOptions} onchange={handleCityChange}
                disabled={isCityDisabled}>
            </lightning-combobox>
            <template if:true={city}>
                <p>Country : {country}</p>
                <p>State : {state}</p>
                <p>City : {city}</p>
            </template>
        </div>
    </lightning-card>
</template>
```

---

## JavaScript Controller  
`dependantPicklistComp.js`

```javascript
import { LightningElement, wire, track } from 'lwc';
import getLocation from '@salesforce/apex/locationController.getLocation';

export default class DependantPicklistComp extends LightningElement {
    @track allData;
    @track countryOptions = [];
    @track stateOptions = [];
    @track cityOptions = [];
    @track country = '';
    @track state = '';
    @track city = '';

    @wire(getLocation)
    wireData({ error, data }) {
        console.log('hii')
        console.log('Data:', data);
        console.log('Error:', error);
        if (data) {
            console.log('Data fetched successfully:', data);
            this.allData = data;
            this.countryOptions = Object.keys(data).map(item => ({
                label: item,
                value: item
            }));
            console.log('Country Options:', this.countryOptions);
        }
        else if (error) {
            console.error('Error fetching data:', error);
        }
    }
    
    get isStateDisabled() {
        return !this.country;
    }
    get isCityDisabled() {
        return !this.state;
    }
    handleCountryChange(event) {
        this.country = event.target.value;
        this.stateOptions = Object.keys(this.allData[this.country]).map(item => ({
            label: item,
            value: item
        }));
        this.state = '';
        this.city = '';
        this.cityOptions = [];
    }
    handleStateChange(event) {
        this.state = event.target.value;
        this.cityOptions = this.allData[this.country][this.state].map(item => ({
            label: item,
            value: item
        }));
        this.city = '';
    }
    handleCityChange(event) {
        this.city = event.target.value;
    }
}
```

---

## Apex Controller  
`locationController.cls`

```apex
public with sharing class locationController {
    @AuraEnabled(cacheable=true)
    public static Map<String,Map<String,List<String>>> getLocation(){
        Map<String,Map<String,List<String>>> data = new Map<String,Map<String,List<String>>>();
        data.put('india',new map<String,List<String>>{
            'karnataka'=>new List<String>{'bangalore','mysore','hubli'},
            'tamilnadu'=>new List<String>{'chennai','coimbatore','madurai'}
        });
        data.put('usa',new map<String,List<String>>{
            'california'=>new List<String>{'san francisco','los angeles','san diego'},
            'new york'=>new List<String>{'new york city','buffalo','albany'}
        });
        data.put('uk',new map<String,List<String>>{
            'england'=>new List<String>{'london','manchester','birmingham'},
            'scotland'=>new List<String>{'glasgow','edinburgh','dundee'}    
        });
        data.put('australia',new map<String,List<String>>{
            'new south wales'=>new List<String>{'sydney','wollongong','perth'},
            'queensland'=>new List<String>{'brisbane','gold coast','sunshine coast'}
        });
        return data;
    }
}
```

---

# 📷 Component Preview

![Dependent Picklist LWC](./images/dependent_picklist_lwc.png)