# 🧩 LWC Utility Components Library

A personal collection of reusable **Lightning Web Components (LWC)** that can be quickly integrated into Salesforce projects without rewriting code from scratch.

This documentation will grow over time as more reusable components are added.

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
