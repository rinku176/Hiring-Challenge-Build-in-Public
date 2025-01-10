## Objective
Build a modular frontend for an **Interactive Worksheet** with the ability to create and interact with worksheets. Additionally, create a backend API to manage worksheets and provide feedback for user responses. The app should include authentication using Firebase to ensure secure access and role-based permissions.

This task will test your ability to:
- Build reusable and interactive frontend components using Vue.js.
- Integrate Firebase Authentication.
- Create a structured and well-documented FastAPI backend.
- Integrate Firestore
- Work with modular architecture and routing in both frontend and backend.

---

### Frontend (Vue.js)

1. **Authentication**:
   - Integrate Firebase Authentication to support email/password or sign-in with Google login and registration.
   - Display a login form and a registration form.
   - Implement logout functionality.
   - Use Vue Router guards to protect routes based on authentication status and roles.

2. **Interactive Worksheet Page**:
   - Display a list of worksheets available for the user.
   - Clicking on a worksheet should show the relevant worksheet with questions and answer options.
   - Each worksheet should display sentences with blanks or dropdowns where users can select answers.
   - Show feedback after the user submits answers (e.g., correct/incorrect).

3. **Worksheet Builder Page**:
   - Accessible only to users with an **admin role**.
   - A form to create new worksheets with the following fields:
     - Worksheet Title.
     - Sentences with placeholders for answers.
     - Options for correct answers (e.g., Dropdown values like Noun, Verb, Adjective).

4. **General Requirements**:
   - Use Vue Router for navigation between the Interactive Worksheet and Worksheet Builder pages.
   - Use Pinia or Vuex for state management.
   - Use firestore for persistence 
   - Ensure a clean and responsive UI (CSS or a utility framework like Tailwind).
   - Follow modular component design principles.

### Evaluation Criteria
- **Frontend**:
  - Modular components and code clarity.
  - Integration of Firebase Authentication.
  - Interactivity and correctness of the worksheet.
  - Use of state management and routing.

- **Backend**:
  - API correctness and adherence to specifications.
  - Proper use of FastAPI and Pydantic.
  - Validation of Firebase tokens.
  - Basic test coverage.

- **Overall**:
  - Commit history and code organization.
  - Clarity and completeness of the README.
  - Creativity in the worksheet design.

### Bonus Points
1. Dockerize the frontend and backend.
2. Add unit tests for key frontend components.

---

## Submission Guidelines
1. Fork this repository.
2. Complete the task by adding code to the `frontend/` and `backend/` directories.
3. Push your changes to your forked repository.
4. Create a pull request to this repository with the following:
   - A short description of your implementation.
   - Any assumptions or design decisions you made.

---


## Repository Structure
```plaintext
interactive-worksheet-task/
├── frontend/          # Vue.js app
├── backend/           # FastAPI app
├── README.md          # Task instructions
└── .gitignore         # Ignore unnecessary files
```

---

### Example Worksheet
Title: **Complete the Sentences: Parts of Speech Worksheet**

Questions:

1. **Sentence**: "She quickly _____."
   - **Options**: Runs, Ran, Running
   - **Correct Answer**: Runs

2. **Sentence**: "They are playing in the _____."
   - **Options**: Park, Quickly, Walked
   - **Correct Answer**: Park

---
