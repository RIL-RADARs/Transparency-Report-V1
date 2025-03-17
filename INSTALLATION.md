![image](https://github.com/user-attachments/assets/39e4ba04-3ce5-4b8a-97ae-82b0fdaf60c5)# Installation Guide

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

     We used custom code from Memberstack. You can find detailed instructions [here](https://www.memberstack.com/scripts/custom-multi-selects). Configure the choice values in "Custom attributes", add Name as "ms-code-select-options", Value in format like "Commercial License,Proprietary,Public Domain,Fair Use Claim,Other", seperating each choice with a single ",".
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
   - RADARs used two projects to store **Accessment Score** and **Report Generation**.
      ![image](https://github.com/user-attachments/assets/034f37e4-1914-4386-9233-c4d9943ffe2e)
3. Set rules to allow authenticated users to read/write.
   - For **Accessment Score**, the rules are:
      ```firestore-rules
      rules_version = '2';
      service cloud.firestore {
        match /databases/{database}/documents {
          
          // Allows any user (with session-based ID) to write responses
          match /users/{sessionId}/responses/{sectionId} {
            allow read, write;
          }
          
          // Benchmark scores (highest possible section scores)
          match /sections/{sectionId} {
            allow read; // Everyone can read max scores
            allow write;
          }
        }
      }
      ```
   - For **Report Generation**, we didn't set any specific rules.
     
4. Connect **app** with project.
   We connected Zapier with the **Report Generation** project for automation.
   ![image](https://github.com/user-attachments/assets/4428091b-e041-4455-9d92-d29faa34294e)

5. Configure **Functions** when needed.
   For project **Report Generation**, we set up two functions:
   ![image](https://github.com/user-attachments/assets/e8eedf68-6687-45b8-857b-ca1acbef9379)
   
   1️⃣ **Get Report Data**  
   - The **`getReportData` function** is a Firebase Cloud Function that retrieves a generated AI transparency report based on a given **slug** (unique identifier for a report). It queries Firestore and returns relevant report metadata.
   - ### 🛠️ **What This Function Does**
      1. Accepts a **slug** as a query parameter (`?slug=your-report-slug`).
      2. Searches for a document in Firestore under the `Response_Status` collection where `slug` matches.
      3. If no matching document is found, it returns `{ error: "Document not found" }`.
      4. If a report exists, it retrieves:
         - `webflowItemId` (for Webflow integration).
         - `userEmail` (associated with the report).
         - `slug` (report identifier).
      5. Implements **CORS headers** to allow secure communication between Webflow and Firebase.
   - #### 🛠️ **Add getReportData in `index.js` **
     ![image](https://github.com/user-attachments/assets/777ceaf5-9ed2-4f38-a7a0-3bf6010a17b6)
     In your `functions/index.js`, add the following function:
      ```
      /**
       * Import function triggers from their respective submodules:
       *
       * const {onCall} = require("firebase-functions/v2/https");
       * const {onDocumentWritten} = require("firebase-functions/v2/firestore");
       *
       * See a full list of supported triggers at https://firebase.google.com/docs/functions
       */
      
      /*
      const functions = require("firebase-functions");
      const admin = require("firebase-admin");
      const cors = require("cors")({
        origin: ["https://brightpath-report.webflow.io", "https://www.brightpath-report.webflow.io"],
        credentials: true,
      });
      
      admin.initializeApp();
      
      // checkStatus 函数
      exports.checkStatus = functions.https.onRequest(async (req, res) => {
        return cors(req, res, async () => {
          res.set("Access-Control-Allow-Origin", "https://brightpath-report.webflow.io");
          res.set("Access-Control-Allow-Credentials", "true");
      
          if (req.method === "OPTIONS") {
            res.set("Access-Control-Allow-Methods", "GET");
            res.set("Access-Control-Allow-Headers", "Content-Type,Accept");
            res.status(204).send("");
            return;
          }
      
          const email = req.query.email;
          if (!email) {
            return res.json({
              exists: false,
              completed: false,
            });
          }
      
          try {
            const docRef = admin.firestore()
                .collection("Response_Status")
                .doc(email);
      
            const docSnap = await docRef.get();
      
            if (!docSnap.exists) {
              return res.json({
                exists: false,
                completed: false,
              });
            } else {
              const data = docSnap.data();
              if (data.completed === "true" && data.Item_ID) {
                return res.json({
                  exists: true,
                  completed: true,
                  slug: data.slug || "",
                  webflowItemId: data.Item_ID,
                });
              } else {
                return res.json({
                  exists: true,
                  completed: false,
                });
              }
            }
          } catch (error) {
            console.error("Error checking status:", error);
            return res.status(500).json({
              error: "Internal server error",
              details: error.message,
            });
          }
        });
      });
      
      // getReportData 函数
      exports.getReportData = functions.https.onRequest(async (req, res) => {
        cors(req, res, async () => {
          const slug = req.query.slug;
          if (!slug) {
            return res.status(400).json({error: "Slug is required"});
          }
      
          try {
            const snapshot = await admin.firestore()
                .collection("Response_Status")
                .where("slug", "==", slug)
                .limit(1)
                .get();
      
            if (snapshot.empty) {
              return res.status(404).json({error: "Document not found"});
            }
      
            const doc = snapshot.docs[0];
            const data = doc.data();
      
            return res.json({
              webflowItemId: data.Item_ID,
              userEmail: doc.id,
              completed: data.completed === "true",
              slug: data.slug,
            });
          } catch (error) {
            console.error("Error:", error);
            return res.status(500).json({
              error: "Server error",
              details: error.message,
            });
          }
        });
      });
       */
      const functions = require("firebase-functions");
      const admin = require("firebase-admin");
      const cors = require("cors")({origin: true});
      
      admin.initializeApp();
      
      // checkStatus 函数
      exports.checkStatus = functions.https.onRequest(async (req, res) => {
        // 设置允许所有来源的访问
        res.set("Access-Control-Allow-Origin", "*");
        res.set("Access-Control-Allow-Methods", "GET, OPTIONS");
        res.set("Access-Control-Allow-Headers", "Content-Type");
      
        if (req.method === "OPTIONS") {
          res.status(204).send("");
          return;
        }
      
        const email = req.query.email;
        if (!email) {
          return res.json({exists: false});
        }
      
        try {
          const docRef = admin.firestore().collection("Response_Status").doc(email);
          const docSnap = await docRef.get();
      
          if (!docSnap.exists) {
            return res.json({exists: false});
          }
      
          const data = docSnap.data();
          return res.json({
            exists: true,
            slug: data.slug || "",
            webflowItemId: data.Item_ID || "",
          });
        } catch (error) {
          console.error("Error checking status:", error);
          return res.status(500).json({
            error: "Internal server error",
            details: error.message,
          });
        }
      });
      
      // getReportData 函数
      exports.getReportData = functions.https.onRequest(async (req, res) => {
        cors(req, res, async () => {
          const slug = req.query.slug;
          if (!slug) {
            return res.status(400).json({error: "Slug is required"});
          }
      
          try {
            const snapshot = await admin.firestore()
                .collection("Response_Status")
                .where("slug", "==", slug)
                .limit(1)
                .get();
      
            if (snapshot.empty) {
              return res.status(404).json({error: "Document not found"});
            }
      
            const doc = snapshot.docs[0];
            const data = doc.data();
      
            return res.json({
              webflowItemId: data.Item_ID,
              userEmail: doc.id,
              slug: data.slug,
            });
          } catch (error) {
            console.error("Error:", error);
            return res.status(500).json({
              error: "Server error",
              details: error.message,
            });
          }
        });
      });
      
      ``` 

   2️⃣ **Check Report Generation Status**  
   - The **`checkStatus` function** is a Firebase Cloud Function used to **verify if a report has been generated** for a given user email. It interacts with Firestore to check the status of a user's response and returns relevant metadata.
   - #### 🛠️ **What This Function Does**
      1. Accepts an **email** as a query parameter.
      2. Looks for the **corresponding document** in Firestore under `Response_Status/{email}`.
      3. If no document is found, it returns `{ exists: false }`.
      4. If the document exists:
         - Retrieves the **slug** and **Webflow item ID**.
         - Returns `{ exists: true, slug, webflowItemId }`.
      5. Implements **CORS headers** to allow secure communication between Webflow and Firebase.
   - #### 🛠️ **Add checkStatus in `index.js` **
     ![image](https://github.com/user-attachments/assets/db2f46a2-d6fc-49e0-81e7-42e8c05a49f9)
     In your `functions/index.js`, add the following function:
      ```
      /**
       * Import function triggers from their respective submodules:
       *
       * const {onCall} = require("firebase-functions/v2/https");
       * const {onDocumentWritten} = require("firebase-functions/v2/firestore");
       *
       * See a full list of supported triggers at https://firebase.google.com/docs/functions
       */
      
      /*
      const functions = require("firebase-functions");
      const admin = require("firebase-admin");
      const cors = require("cors")({
        origin: ["https://brightpath-report.webflow.io", "https://www.brightpath-report.webflow.io"],
        credentials: true,
      });
      
      admin.initializeApp();
      
      // checkStatus 函数
      exports.checkStatus = functions.https.onRequest(async (req, res) => {
        return cors(req, res, async () => {
          res.set("Access-Control-Allow-Origin", "https://brightpath-report.webflow.io");
          res.set("Access-Control-Allow-Credentials", "true");
      
          if (req.method === "OPTIONS") {
            res.set("Access-Control-Allow-Methods", "GET");
            res.set("Access-Control-Allow-Headers", "Content-Type,Accept");
            res.status(204).send("");
            return;
          }
      
          const email = req.query.email;
          if (!email) {
            return res.json({
              exists: false,
              completed: false,
            });
          }
      
          try {
            const docRef = admin.firestore()
                .collection("Response_Status")
                .doc(email);
      
            const docSnap = await docRef.get();
      
            if (!docSnap.exists) {
              return res.json({
                exists: false,
                completed: false,
              });
            } else {
              const data = docSnap.data();
              if (data.completed === "true" && data.Item_ID) {
                return res.json({
                  exists: true,
                  completed: true,
                  slug: data.slug || "",
                  webflowItemId: data.Item_ID,
                });
              } else {
                return res.json({
                  exists: true,
                  completed: false,
                });
              }
            }
          } catch (error) {
            console.error("Error checking status:", error);
            return res.status(500).json({
              error: "Internal server error",
              details: error.message,
            });
          }
        });
      });
      
      // getReportData 函数
      exports.getReportData = functions.https.onRequest(async (req, res) => {
        cors(req, res, async () => {
          const slug = req.query.slug;
          if (!slug) {
            return res.status(400).json({error: "Slug is required"});
          }
      
          try {
            const snapshot = await admin.firestore()
                .collection("Response_Status")
                .where("slug", "==", slug)
                .limit(1)
                .get();
      
            if (snapshot.empty) {
              return res.status(404).json({error: "Document not found"});
            }
      
            const doc = snapshot.docs[0];
            const data = doc.data();
      
            return res.json({
              webflowItemId: data.Item_ID,
              userEmail: doc.id,
              completed: data.completed === "true",
              slug: data.slug,
            });
          } catch (error) {
            console.error("Error:", error);
            return res.status(500).json({
              error: "Server error",
              details: error.message,
            });
          }
        });
      });
       */
      const functions = require("firebase-functions");
      const admin = require("firebase-admin");
      const cors = require("cors")({origin: true});
      
      admin.initializeApp();
      
      // checkStatus 函数
      exports.checkStatus = functions.https.onRequest(async (req, res) => {
        // 设置允许所有来源的访问
        res.set("Access-Control-Allow-Origin", "*");
        res.set("Access-Control-Allow-Methods", "GET, OPTIONS");
        res.set("Access-Control-Allow-Headers", "Content-Type");
      
        if (req.method === "OPTIONS") {
          res.status(204).send("");
          return;
        }
      
        const email = req.query.email;
        if (!email) {
          return res.json({exists: false});
        }
      
        try {
          const docRef = admin.firestore().collection("Response_Status").doc(email);
          const docSnap = await docRef.get();
      
          if (!docSnap.exists) {
            return res.json({exists: false});
          }
      
          const data = docSnap.data();
          return res.json({
            exists: true,
            slug: data.slug || "",
            webflowItemId: data.Item_ID || "",
          });
        } catch (error) {
          console.error("Error checking status:", error);
          return res.status(500).json({
            error: "Internal server error",
            details: error.message,
          });
        }
      });
      
      // getReportData 函数
      exports.getReportData = functions.https.onRequest(async (req, res) => {
        cors(req, res, async () => {
          const slug = req.query.slug;
          if (!slug) {
            return res.status(400).json({error: "Slug is required"});
          }
      
          try {
            const snapshot = await admin.firestore()
                .collection("Response_Status")
                .where("slug", "==", slug)
                .limit(1)
                .get();
      
            if (snapshot.empty) {
              return res.status(404).json({error: "Document not found"});
            }
      
            const doc = snapshot.docs[0];
            const data = doc.data();
      
            return res.json({
              webflowItemId: data.Item_ID,
              userEmail: doc.id,
              slug: data.slug,
            });
          } catch (error) {
            console.error("Error:", error);
            return res.status(500).json({
              error: "Server error",
              details: error.message,
            });
          }
        });
      });
      ```
     
7. Copy Firebase configuration and store it in a `.env` file (if applicable).

### **Step 3: Automate Data Processing with Zapier**
Zapier automates the connection between Webflow, Firebase, and AI processing.

1. Sign up for [Zapier](https://zapier.com/).
2. **Create the following Zaps**:
   - **Trigger**: Webflow form submission
   - **Action**: Store user inputs in Firebase
   - **Action**: Call OpenAI API to generate report
   - **Action**: Save report data back into Firebase/Webflow CMS
3. Test your Zapier workflows.

### **Step 4: Build a survey flow**
RADARs uses **Inputflow** to create dynamic survey flows and branching logic.
![image](https://github.com/user-attachments/assets/b207e0bb-1c19-4cd3-a056-d0c414aab601)
#### 🛠️ **Setting Up Inputflow in Webflow**
1. **Install the Inputflow Plugin**  
   - In the Webflow site, go to **Apps**.  
   - Search **Inputflow** and add it to your project.  
   - Alternatively, install from [Inputflow’s official site](https://www.inputflow.io/).
     ![image](https://github.com/user-attachments/assets/a2724818-54ed-46da-9748-2b2c9d24c4df)

2. **Create a New Survey Flow**  
   - Configure input types (**text, multiple choice, dropdowns, etc.**).  
   - Open the **Inputflow dashboard** inside Webflow.  
   - Click **"New Survey"** and define a **starting question**.  

3. **Set Up Branching Logic**  
   - Add **conditional paths** based on user responses.  
   - Example:  
     - If **Industry = Healthcare**, show **Healthcare-specific AI questions**.  
     - If **Industry = Finance**, redirect to **Finance-related AI risk questions**.  
   - Use **"Conditional Logic"** settings to guide users dynamically.
     ![image](https://github.com/user-attachments/assets/26eece9a-9bcc-468e-9d85-3b3c402c3cc6)

4. **Connect Inputflow Data to Firebase**  
   - Ensure Webflow form submissions store responses in **Webflow CMS**.  
   - Use **Zapier** to send collected data to **Firebase Firestore**.  
   - Test by filling out a form and verifying data storage.  

5. **Final Check:** Preview the survey, validate navigation between steps, and ensure responses trigger the right follow-up questions. 

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
