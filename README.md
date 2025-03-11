# RADARs: Responsible AI Disclosure + Reporting System

## Overview

RADARs is a web app that streamlines AI transparency reporting for early-stage startups. By combining smart templates with real-time AI guidance, our tool helps startups align with industry best practices, identify risks, get actioanble suggestions, and communicate transparency efforts effectively to stakeholders.

## Getting Started

### Prerequisites

Ensure you have:

- A modern web browser (Chrome, Firefox, Edge)
- An internet connection
- Necessary API access (if applicable)

### Installation

1. Clone the repository:
   ```sh
   git clone https://github.com/your-group-name/radars-ai-transparency.git
   ```
2. Navigate to the project directory:
   ```sh
   cd radars-ai-transparency
   ```
3. Install dependencies (if applicable):
   ```sh
   npm install  # or pip install -r requirements.txt
   ```

## Usage

### Step 1: Access the Prototype

- Visit the live prototype here: [https://brightpath-report.webflow.io/ ](https://brightpath-report.webflow.io/ )

### Step 2: Enter Basic Information

- Provide details about your **business domain, AI base model, use cases, and target audience**.

### Step 3: Complete the Tailored Questionnaire

- Answer a set of questions related to AI practices, including **data handling, testing methods, and ethical considerations**.

### Step 4: Generate the Transparency Report

- Click **Generate Report** to produce a comprehensive document containing:
  - Identified risks and prioritization
  - Severity levels
  - Actionable suggestions tailored to your industry
  - A comprehensive transparency report summarizing your current AI practices

## Key Features

- **🔍 Customized Recommendations** – Focus only on the risks relevant to your industry and audience.
- **🧠 AI-Powered Guidance** – Get real-time assistance with AI-driven insights.
- **⚠️ Priority Risk Assessment** – Receive structured evaluation reports with actionable suggestions.

## Technical Stack

- **Frontend:** Webflow
- **Backend:** Firebase Firestore
- **Automation:** Zapier
- **AI Processing:** OpenAI, Claude API

## Building Your Own Version

If you want to customize or extend this project for your startup's specific needs, follow these steps:

### 1. Set Up Your Own Firebase Database
- Create a Firebase project and enable Firestore.
- Update the Firebase configuration in the code (provide file path and details).
- Set up necessary authentication rules.

### 2. Configure Zapier Automations
- Set up Zapier to automate report generation and data processing.
- Modify the **Zap Flows** to fit your specific use case.

### 3. Customize Webflow CMS
- Duplicate the Webflow project and adjust the design to match your branding.
- Link Webflow to your Firebase database.

### 4. Modify AI Processing
- The system uses OpenAI & Claude API to generate reports.
- If needed, update the prompts or switch to a different LLM provider.

### 5. Deploy Your Version
- Host on **Vercel, Netlify, or Firebase Hosting**.
- Update environment variables for API keys and database connections.

## Troubleshooting

- **UI Display Issues** – Ensure browser zoom is at 100%.
- **Slow Report Generation** – Allow up to 2 minutes; refresh if delays persist.
- **API Errors** – Verify API key and internet connection.

## Contribution

We welcome contributions! To contribute:

1. Fork the repository.
2. Create a new branch: `git checkout -b feature/your-feature-name`
3. Commit changes: `git commit -m "Add new feature"`
4. Push to the branch: `git push origin feature/your-feature-name`
5. Open a pull request.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Feedback & Support

For feedback or support, please contact our team or submit an issue in the repository.

---

⭐ If you find this project useful, give us a star on GitHub! ⭐

