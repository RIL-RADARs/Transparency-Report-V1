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
6. Copy Firebase configuration and store it in a `.env` file (if applicable).

### **Step 3: Automate Data Processing with Zapier**
Zapier automates the connection between **Webflow, Firebase, and AI processing**. This allows user-submitted data to flow through the system and generate AI transparency reports.

#### **Prerequisites**
- A **Zapier** account ([Sign up here](https://zapier.com/))
- A **Webflow** project with a form connected
- A **Firebase Firestore database** set up

---

1. Sign up for [Zapier](https://zapier.com/).
2. **Create the following Zaps**:
   - 1 **Trigger**: Webflow Form Submission
      1. **Go to Zapier** and create a new **Zap**.
      2. Select **"Webhooks by Zapier"** as the trigger app.
      3. Choose **"Catch Hook"** as the trigger event.
      4. Copy the **Webhook URL** provided by Zapier.
      ![image](https://github.com/user-attachments/assets/feb472d3-99ac-43c5-bcbb-16e2f3e6b8a3) 
      5. **Go to Webflow** and navigate to the form settings.
      6. In the **"Form Block Settings"** section "Connections", paste the **Zapier Webhook URL** in "Action".
      ![image](https://github.com/user-attachments/assets/1646d26d-0a0f-4365-b363-23bb2215a629)
      7. Set the method to **POST**.
      8. **Save & Publish** your Webflow project.
      9. Return to Zapier and **test the trigger** by submitting the Webflow form.
      📌 **Expected Outcome**: When a user submits the form in Webflow, Zapier captures the data.
   - 2 **Action**: Code by Zapier
     This step **generates a unique slug** based on the user's name. The slug is used as a unique identifier for reports and database records.
     ![image](https://github.com/user-attachments/assets/3c52369e-f79d-44d5-ba2e-c291dd34eb78)
      1. **In Zapier**, add a new action.
      2. Select **"Code by Zapier"** as the action app.
      3. Choose **"Run JavaScript"** as the action event.
      4. Click **Continue** to configure the code step.
      5. Under **Input Data**, set the following:
         - **Key:** `userName`
         - **Value:** Select the user's name from the Webflow form submission (Step 1).
      6. **In the code editor**, paste the following JavaScript code:
          ```
         // 1) Get user input (e.g., userName = "Jane"):
         let userName = inputData.userName;  // For example, "Jane"
         
         // 2) Perform slugify (convert spaces to "-", lowercase, etc.)
         function slugify(str) {
           return str
             .toLowerCase()
             .replace(/[^a-z0-9]+/g, '-')  // Convert non-alphanumeric characters to "-"
             .replace(/^-|-$/g, '')       // Remove leading and trailing "-"
             .substring(0, 30);           // Limit to a maximum of 30 characters
         }
         
         let baseSlug = slugify(userName); // "jane"
         
         // 3) To ensure uniqueness, add a random suffix
         let random = Math.floor(Math.random() * 1000); // 0~999
         let finalSlug = baseSlug + "-" + random; // "jane-543"
         
         // Output
         return { finalSlug };
         
          ```
      8. Click **Continue**, then **Test & Review** to ensure the function works correctly.
         
   - 3 **Action**: Webflow CMS - Create Live Item
     This step **sends user input data** (including the generated slug) to **Webflow CMS** to store user reports.
      1. **In Zapier**, add a new action.
      2. Select **"Webflow"** as the app.
      3. Choose **"Create Live Item"** as the action event.
      4. Click **Continue** to configure the setup.
      5. **Connect your Webflow account** (if not already connected).
      6. Under **"Site"**, select the **Webflow project** where the reports will be stored.
      7. Under **"Collection"**, select the correct **Webflow CMS Collection** (e.g., "Back-ends" in your setup).
      ![image](https://github.com/user-attachments/assets/dfdefe02-6128-4ceb-a889-30f0a01c8d91)
      **📌 Test and Publish**
         1. Click **Test & Review** to verify that the data is correctly sent to Webflow.
         2. Check **Webflow CMS** to confirm the new entry appears.
         3. Click **Continue** and publish the Zap.
    
   - 4 **Action**: Firebase Firestore - Create Document
     This step **saves user responses** from Webflow into **Firebase Firestore** for storage and later report generation.
      1. **In Zapier**, add a new action.
      2. Select **"Firebase Firestore"** as the app.
      3. Choose **"Create Cloud Firestore Document"** as the action event.
      4. Click **Continue** to configure the setup.
      5. **Connect your Firebase account** (if not already connected).
      6. Under **"Collection"**, enter the Firestore collection where the data should be stored:  
         ✅ Example: `Question_Answer`
      7. Under **"Document ID"**, set a unique identifier for each user submission:  
         ✅ Example: Use **User Email** (`testingfelicity01@gmail.com`)  
         This ensures each submission is stored under the corresponding user’s document.
      ![image](https://github.com/user-attachments/assets/e0feae73-54b1-49b7-a34e-12d2d3a96fc6)
      8. Under **"Data"**, map all user responses you need to Firestore fields.
      9. Click **Test & Review** to verify that the data is correctly stored in Firestore.
      ![image](https://github.com/user-attachments/assets/efd45149-4ffa-4530-8be6-91cee4f059ff)

 - 5 **Action**: API Call - Generate AI Transparency Report
     This step sends user responses to **Claude AI** to generate a structured **AI transparency report**. Here's the example of Anthropic (Claude) API call.
      ![image](https://github.com/user-attachments/assets/d272ebab-8854-4d88-9c7f-e440c87e1ea1)
      1. **In Zapier**, add a new action.
      2. Select **"Anthropic (Claude)"** as the app.
      3. Choose **"Send Message"** as the action event.
      4. Connect your **Claude AI account**.
      5. **Configure the request:**
         - Use the user’s responses (from Firebase) as input.
         - Ensure the output is in **structured JSON format**.
         - Include instructions to **strictly follow user input** and **not generate assumptions**.
      6. **Run a test** to confirm Claude generates a valid report.
      7. **Check the output** for proper formatting.
      8. **Publish the Zap** to automate report generation.

 - 6 **Action**: 
     This step **ensures the AI-generated report accurately reflects user input** while improving professionalism and compliance. Here's the example of Anthropic (Claude) API call.
      ![image](https://github.com/user-attachments/assets/2b13cac3-2fe6-48d0-9020-df3a688836ae)
      1. **In Zapier**, add a new action.
      2. Select **"Anthropic (Claude)"** as the app.
      3. Choose **"Send Message"** as the action event.
      4. **Connect your Claude AI account**.
      5. **Configure the request**:
         - Send the **initial AI-generated report** (from Zap 5).
         - Include **user input for reference**.
         - Instruct the AI to **match user input exactly**, correct errors, and enhance clarity.
      6. **Run a test** to confirm Claude refines the report accurately.
      7. **Check the output** to ensure it aligns with user input.
      8. **Publish the Zap** to automate report verification.

 - 7 **Action**: Format AI-Generated Report into JSON
     This step ensures the AI-generated report is correctly structured in JSON format before storing it in Firebase.
      1. **Add a "Code by Zapier" Action**  
         - Select **Run JavaScript** as the action event.
      2. **Process AI Output**  
         - Take the AI-generated response as input.
         - Parse and format it into a valid JSON object.
           ![image](https://github.com/user-attachments/assets/58b8b366-5ef6-4195-bfb5-3487249c3eb5)
         ```
           // Parse the input data
           let rawInput = inputData.json_input;
      
           // Process and format the JSON
           try {
               // Parse the JSON data
               let parsedData = JSON.parse(rawInput);
      
               // Format the JSON with indentation (without adding extra escape characters)
               let formattedJson = JSON.stringify(parsedData, null, 2);
      
               // Return the valid JSON object, not as a string
               return JSON.parse(formattedJson);
      
           } catch (error) {
               // If JSON parsing fails, return an error message
               return { error: "Invalid JSON format" };
           }
         ```          
      3. **Handle Errors**  
         - If the AI response is not in a valid JSON format, return an error message.
      4. Click **Continue**, then **Test & Review** to ensure the JSON is correctly formatted.
         ![image](https://github.com/user-attachments/assets/5f0362a0-c088-42a5-a46b-ef391ac03dbc)

 - 8-10 **Store User-Editable Content in Firebase**:
   This step **saves user-editable sections of the AI-generated report** to Firebase Firestore for structured storage and retrieval.
      ![image](https://github.com/user-attachments/assets/3d64d567-6e83-4c85-b6e3-33a56aa27182)
     1. **In Zapier**, add a new action.
     2. Select **"Firebase / Firestore"** as the app.
     3. Choose **"Create Cloud Firestore Document"** as the action event.
     4. **Connect your Firebase account**.
     5. **Configure the request**:
        - Set **Collection** to the relevant section (e.g., `Generated_[Section-Name]`).
        - Use the **user's email** as the Document ID.
        - Map the **necessary fields** (e.g., Industry, Product Details, Transparency Info) from the structured AI-generated report.
     6. **Run a test** to confirm that the document is successfully created in Firestore.
     7. **Check the output** to ensure it contains the correct user-editable content.
     8. **Publish the Zap** to store and manage the editable report sections dynamically.

 - 11 **Action**: Calculate Scores
     This step analyzes user responses, extracts relevant sections, and assigns risk scores based on predefined criteria, helping evaluate compliance levels dynamically. It categorizes risks as High, Medium, or Low and structures the results for further processing or storage.
     1. **In Zapier**, add a new action.
     2. Select **"Code by Zapier"** as the app.
     3. Choose **"Run JavaScript"** as the action event.
     4. **Click Continue** to configure the code step.
     5. Under **Input Data**, set:
        - **User ID:** Retrieve from user responses.
        - **Relevant Fields:** Map compliance-related answers (e.g., privacy, security, data-sharing).
      ![image](https://github.com/user-attachments/assets/cf3feaa8-0832-4ac2-8c29-a63042cc9e7c)
     6. **In the code editor**, paste the following JavaScript code:
      ```
      let sectionScores = {};
      let sectionSeverity = {};  
      let sectionNames = {};  
      let userId = inputData["user_id"] || "unknown_user"; 
      
      console.log("Received data:", JSON.stringify(inputData, null, 2));
      
      for (let key in inputData) {
          if (key === "user_id" || !inputData[key] || inputData[key].includes("--placeholder--")) continue; 
      
          let value = inputData[key];
          console.log("Processing field:", key, "Value:", value); 
      
          // Extract section name and question number (e.g., "Privacy & Security_2_2")
          let sectionMatch = key.match(/Case \d+ ([^_]+)_\d+_\d+/);
          if (sectionMatch) {
              let sectionName = sectionMatch[1];  // Extracted section name (e.g., "Privacy & Security")
      
              // Ensure section exists in the tracking objects
              if (!sectionScores[sectionName]) {
                  sectionScores[sectionName] = 0;
                  sectionSeverity[sectionName] = "Medium";  
                  sectionNames[sectionName] = sectionName;
              }
      
              // Extract score (if "|1" exists)
              let scoreMatch = value.match(/\|(\d+)$/);
              if (scoreMatch) {
                  let score = parseInt(scoreMatch[1], 10);
                  sectionScores[sectionName] += score;  
              }
      
              // Detect High Risk (Improved method)
              if (value.toLowerCase().includes("high risk")) {
                  sectionSeverity[sectionName] = "High";  
              }
          }
      }
      
      // **Ensure all sections have correct severity levels**
      for (let section in sectionScores) {
          if (sectionScores[section] === 0) {
              sectionSeverity[section] = "Low";  // If no bad practices are detected, set to "Low"
          } else if (!sectionSeverity[section]) {
              sectionSeverity[section] = "Medium";  // Default to "Medium" if no "High Risk" found
          }
      }
      
      // Output result formatted for Firebase
      output = {
          user_id: userId,
          section_scores: sectionScores,
          section_severity: sectionSeverity,
          section_names: sectionNames
      };
      ```
     7. **Click Continue**, then **Test & Review** to verify the scoring process.
     8. **Check the output**:
        - Scores are assigned to each section.
        - Severity levels (Low, Medium, High) are correctly categorized.
      ![image](https://github.com/user-attachments/assets/8ffaf555-9fbc-457d-85e4-7dba84d4fbd8)
     9. **Publish the Zap** to automate the risk scoring process.

 - 12 **Action**: Store Scores
     This step **saves the calculated scores for each section into Firebase Firestore** for future reference and further analysis.
      1. **In Zapier**, add a new action.  
      2. Select **"Firebase / Firestore"** as the action app.  
      3. Choose **"Create Cloud Firestore Document"** as the action event.  
      4. **Connect your Firebase Firestore account**.  
      5. **Configure the request** containing **Collection:** `User Scores` (stores the section scores), **Document ID:** `user_id` (ensures scores are linked to the correct user), and **Data Fields:**  
      ![image](https://github.com/user-attachments/assets/6a01809f-fc59-47c2-aa92-283fb918afff)
      6. **Run a test** to confirm the scores are correctly stored in Firestore.
      ![image](https://github.com/user-attachments/assets/5d7a0523-2630-47df-bdc7-3193f29a8b3b)

 - 13 **Action**: Match Practice with Business Risks Using Existing Responsible AI frameworks (Microsoft Responsible AI Standard)
     This step **compares user responses about AI practices with Microsoft's Responsible AI Standards** to identify potential risks and alignment gaps. The output helps generate tailored recommendations.
     1. **In Zapier**, add a new action.  
     2. Select **"Code by Zapier"** as the action app.  
     3. Choose **"Run Python"** as the action event.  
     4. Click **Continue** to configure the code step.  
     5. Under **Input Data**, set the following:  
        - **Key:** `user_input`  
        - **Value:** Select user responses from previous steps related to AI practices.
        - **Key:** `markdown_text`  
        - **Value:** Microsoft's Responsible AI Standard content.  
         ![image](https://github.com/user-attachments/assets/32676929-f0d6-4e4a-90e4-4a35162a0136)
     6. **In the code editor**, process the user responses:  
        - Extract relevant responses regarding data privacy, bias mitigation, and AI risk factors. 
        - Match these responses against Microsoft's Responsible AI Standard documentation.  
        - Identify risk areas and gaps in compliance.  
        - Format output as structured Markdown text.
         ```
         import re
         import json
         
         def parse_user_input_lines(user_input_raw):
             """
             Parse multi-line string into question->answer pairs. If it contains |1, record it as bad_practice.
             The regex assumes each line is in a format like:
               2.1. Some question ...?  {{someAnswer}} extra
             If the format differs, the pattern may need adjustments.
             """
             user_dict = {}
             bad_practices = {}
             
             pattern = re.compile(r'^(.*?)\{\{(.*?)\}\}(.*)$')  
             # Split into 3 groups:
             # (1) question_part => (.*?) up to {{
             # (2) braces => (.*?) inside the braces
             # (3) remainder => (.*)$ possibly containing |1
         
             lines = user_input_raw.splitlines()
             line_index = 0
             for line in lines:
                 line_index += 1
                 line = line.strip()
                 if not line or line.startswith("-----"):
                     # Skip empty lines & "----- Start of case..."
                     continue
                 
                 match = pattern.match(line)
                 if match:
                     question_part = match.group(1).strip()
                     inside_braces = match.group(2).strip()
                     remainder = match.group(3).strip()
                     raw_answer = (inside_braces + " " + remainder).strip()
                 else:
                     question_part = f"Line{line_index}"
                     raw_answer = line
                 
                 # Check for |1
                 is_bad = ("|1" in line)
                 clean_answer = raw_answer.replace("|1", "").strip()
         
                 # Store question->answer pair
                 key = f"Line{line_index}: {question_part}"
                 user_dict[key] = clean_answer
         
                 if is_bad:
                     bad_practices[key] = clean_answer
             
             return user_dict, bad_practices
         
         def approx_token_count(text):
             """
             Simple approximation: In common English/ASCII text, 1 token ~ 3-4 characters.
             Here, we assume every 4 characters ~ 1 token.
             """
             return int(len(text) / 4) + 1
         
         def main(input_data):
             # 1) Read the input passed from Zapier
             user_input_raw = input_data.get("user_input", "").strip()
             markdown_text = input_data.get("markdown_text", "")
         
             print("DEBUG: user_input_raw =", repr(user_input_raw))
         
             # 2) Parse multi-line input => user_dict, bad_practices
             user_dict, bad_practices = parse_user_input_lines(user_input_raw)
             print("DEBUG: user_dict =", user_dict)
             print("DEBUG: bad_practices =", bad_practices)
         
             # 3) Define keywords to search in markdown_text
             keywords = {
                 "PII": ["personal identifiable information", "PII", "user data", "sensitive data"],
                 "Data Sharing": ["third party", "vendor", "data sharing", "external AI", "uploaded files"],
                 "AI Fairness": ["bias", "fairness", "discrimination", "ethics"],
                 "Transparency": ["explainability", "decision making", "black box", "accountability"],
                 "HIGH RISK": ["High Risk"],
                 "MEDIUM RISK": ["Medium Risk"]
             }
         
             # 4) Match snippets
             matched_sections = {}
             
             # Token Limit -> you can adjust as needed (Claude 1/2 typical ~ 8k-100k tokens)
             # Here, we use 4000 tokens as a safe limit for demonstration
             MAX_TOKENS = 4000  
             total_tokens_used = 0
         
             for question_key, answer_text in bad_practices.items():
                 matched_sections[question_key] = []
                 
                 # Remove duplicates
                 used_snippets = set()
         
                 for category, word_list in keywords.items():
                     for word in word_list:
                         pattern2 = re.compile(re.escape(word), re.IGNORECASE)
                         matches = [m.start() for m in pattern2.finditer(markdown_text)]
                         
                         for match_start in matches:
                             # Option 1: Shorten snippet (+300 characters)
                             snippet_start = max(0, match_start - 80)
                             snippet_end   = min(len(markdown_text), match_start + 300)
                             snippet = markdown_text[snippet_start:snippet_end]
         
                             # Option 3: Remove duplicates
                             if snippet in used_snippets:
                                 continue
                             used_snippets.add(snippet)
         
                             # Option 5: Token limit
                             snippet_tokens = approx_token_count(snippet)
                             if total_tokens_used + snippet_tokens > MAX_TOKENS:
                                 # If exceeding limit -> break and stop adding snippets
                                 break
                             else:
                                 total_tokens_used += snippet_tokens
         
                             # Append snippet
                             matched_sections[question_key].append({
                                 "category": category,
                                 "keyword": word,
                                 "matched_text": snippet
                             })
                     # Check if token limit is reached
                     if total_tokens_used >= MAX_TOKENS:
                         break
                 if total_tokens_used >= MAX_TOKENS:
                     break
         
             # 5) Construct prompt
             claude_prompt = (
                 "## User's Bad Practices:\n"
                 f"{json.dumps(bad_practices, indent=2)}\n\n"
                 "## Related Microsoft AI Standard Sections:\n"
                 f"{json.dumps(matched_sections, indent=2)}"
             )
         
             # 6) Output results
             output = {
                 "prompt": claude_prompt,
                 "bad_practices": bad_practices,
                 "matched_sections": matched_sections,
                 "parsed_lines": user_dict
             }
             return output
         
         # Zapier code by Zapier entry point
         result = main(input_data)
         return result
         
         ```
     7. Click **Continue**, then **Test & Review** to validate the matching process.  
     8. Check that the output includes matched risks and reference sections from Microsoft’s standard.
      ![image](https://github.com/user-attachments/assets/9600a525-fa25-4a4e-92e1-963800131cd0)

 - 14 **Action**: 
     This step

 - 15 **Action**: 
     This step

 - 16 **Action**: 
     This step

 - 17 **Action**: 
     This step

 - 18 **Action**: 
     This step

 - 19 **Action**: 
     This step 


3. Test your Zapier workflows.

### **Step 4: Build a survey flow**
RADARs uses **Inputflow** to create dynamic survey flows and branching logic.

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
     ![image](https://github.com/user-attachments/assets/b207e0bb-1c19-4cd3-a056-d0c414aab601)

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
