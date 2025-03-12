# Installation Guide

This guide provides step-by-step instructions on how to:
1. **Use the RADARs prototype**
2. **Build a similar Webflow-based web app** for AI transparency reporting

---

## 1️⃣ Access the Live Prototype

You can explore and test the RADARs prototype here:  
👉 **[Live Prototype](https://brightpath-report.webflow.io/)**

### How to Use the Prototype:
1. **Enter Basic Information** – Provide details about your business domain, AI model, and audience.
2. **Complete the Tailored Questionnaire** – Answer questions related to AI practices, such as data handling and ethical risks.
3. **Generate Transparency Report** – Click **Generate Report** to receive a comprehensive AI transparency assessment.

For more details on how the prototype works, check out the [Usage Guide](USAGE.md).

---

## 2️⃣ How to Build Your Own Web App

### **Step 1: Set Up a Webflow Account**
1. Sign up for [Webflow](https://webflow.com/).
2. Create a new project or duplicate our **Webflow template** (if shared).
3. Customize the design to match your brand.

### **Step 2: Set Up a Firebase Database**
RADARs uses **Firebase Firestore** to store user inputs and reports.

1. Go to [Firebase Console](https://console.firebase.google.com/).
2. **Create a Firebase project** and enable Firestore.
3. Set rules to allow authenticated users to read/write.
4. Copy Firebase configuration and store it in a `.env` file (if applicable).

### **Step 3: Automate Data Processing with Zapier**
Zapier automates the connection between Webflow, Firebase, and AI processing.

1. Sign up for [Zapier](https://zapier.com/).
2. **Create the following Zaps**:
   - **Trigger**: Webflow form submission
   - **Action**: Store user inputs in Firebase
   - **Action**: Call OpenAI API to generate report
   - **Action**: Save report data back into Firebase/Webflow CMS
3. Test your Zapier workflows.

### **Step 4: Configure AI Report Generation**
1. The system uses **OpenAI API (or Claude API)** to analyze inputs.
2. If you want to customize AI-generated reports:
   - Modify API prompts in Zapier.
   - Use a different AI model if needed.

### **Step 5: Deploy Your Webflow App**
1. Publish your Webflow project using Webflow’s hosting.
2. If using Firebase:
   ```sh
   firebase deploy
