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

---

## 2️⃣ How to Build Your Own Web App

### **Step 1: Set Up a Webflow Site**
1. Sign up for [Webflow](https://webflow.com/).
2. Create your Workspace, then create a new site.
3. Create **Landing Page**.
   - This page is for users to learn and understand the product.
   - You can showcase any information you see fit.
      ![image](https://github.com/user-attachments/assets/af35cc55-c44b-4163-b793-8176eb529a24)
4. Create **Questionaire Page**.
   - This page is where users can input basic information of their business to get customized recommendations of the most relevant risky areas for their business regarding their AI practices.
   - We used webflow "Form block" to contain all the questions.
     ![image](https://github.com/user-attachments/assets/7184b56d-10ab-4399-a04a-412084889360)
   - Note that you will need to put all your question cards inside a single form block to collect all the user inputs altogether for next steps.
5. Add question cards. You can customize your own survey/question list.
   - **Dropdown Choice**:
     Add "Select" from "Forms" section to the page. Configure your choice texts and values in "Settings".
     ![image](https://github.com/user-attachments/assets/40c4e754-1b72-4a94-947c-55cdcebae8fd)
   - **Fill in Blank**:
     Add "Input" from "Forms" section to the page. Configure the Text Feild Name, Type, Placeholder, and if "Required" in "Settings".
     ![image](https://github.com/user-attachments/assets/6ddac6a5-f8c4-4e5c-af0a-d30d7598769b)
     If longer input from users is needed, use "Text Area" instead. Configure the Textare Name, Placeholder, and if "Required" in "Settings".
     ![image](https://github.com/user-attachments/assets/ed8658da-b718-4c7a-b5d2-cb93c9ed0dfb)
   - **Single Choice**:
     Add "Radio Button" from "Forms" section to the page. Configure the Radio Button "Group Name"(Question Name), Choice Value, and if "Required" in "Settings".
     ![image](https://github.com/user-attachments/assets/9506c50b-ee7a-47c5-861e-86e25fd3da62)
     To change the choice display text, select the corresponding "form_radio-label" from Navigator, and configure Radio Button Label Text in "Settings".
     ![image](https://github.com/user-attachments/assets/588b89e6-841c-471a-a20c-3cd4440e1b9c)
   - **Multi Choice**:

      Add? , Configure the choice values in "Custom attributes", add Name as "ms-code-select-options", Value in format like "Commercial License,Proprietary,Public Domain,Fair Use Claim,Other", seperating each choice with a single ",".
     ![image](https://github.com/user-attachments/assets/0a4b53b5-541c-493b-b703-57a8039cd6e1)
6. Create **Report Page**
   - This page is to display the content of the report to users.
     You can use elements from "Basic", "Typography", and Grid or Columns in "others" to make this page.
     ![image](https://github.com/user-attachments/assets/29e5a971-67a4-4a6d-a089-87e6ac9d87df)
7. Customize the page designs to match your brand.

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

### **Step 4: Configure AI Report**
1. The system uses **OpenAI API (or Claude API)** to analyze inputs.
2. If you want to customize AI-generated reports:
   - Modify API prompts in Zapier.
   - Use a different AI model if needed.

### **Step 5: Deploy Your Webflow App**
1. Publish your Webflow project using Webflow’s hosting.
2. If using Firebase:
   ```sh
   firebase deploy
