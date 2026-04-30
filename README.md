# 📑 Project Log - eSportCal

Stage 1 Report

## 👤 Team, Vision & Norms
- **Project Name** : eSportCal.
- **Team** : Antoine & Ilan.
- **Roles** : Unlike traditional structures, we have decided not to assign fixed technical roles (e.g., dedicated Frontend vs. Backend). Instead, we will operate on a Full-Stack & Peer-Programming model. As aspiring DevOps engineers, we believe it is crucial for both members to master every layers of the application. This ensures better redundancy, shared technical knowledge, and a deeper understanding of the integration challenges between the frontend and the database.
- **Collaboration Tools** :
    - Discord: For daily syncs and instant communication.
    - GitHub: For version control (using Pull Requests to review each other's code).
    - Trello: For task management and tracking progress.
- **Objective** : Develop a website that regroups all e-sport matches worldwide.
- **Team Norms** : Daily stand-ups at 10:00 AM

---

## 🛠️ Technical Stack & Strategic Decisions

| Element | Decision | Technical Justification |
| :--- | :--- | :--- |
| **Platform** | **Web (Desktop-first)** | Focus on the "second-screen" experience for gamers. Web ensures instant deployment and maximum accessibility for the MVP. |
| **Frontend** | **React.js (via Vite.js)** | Modern SPA architecture. Vite provides a fast development cycle, essential for building the dashboard within the 4-week window. |
| **Backend (API)** | **Node.js & Express** | We will build our **own REST API** to handle business logic, process data from PandaScore, and manage secure communication with the database. |
| **Database** | **PostgreSQL** | A robust relational database to manage user profiles, team favorites, and cached match data for performance and persistence. |
| **Data Source** | **PandaScore API** | External industry-standard API used as our primary data source for professional schedules and team metadata. |
| **Styling** | **Tailwind CSS** | Utility-first CSS framework to ensure a professional, consistent UI across the entire dashboard and authentication views. |
| **Philosophy** | **Full-Stack Ownership** | By building our own Backend and Database layer, we ensure full control over the system architecture, facilitating future DevOps implementations (Docker, CI/CD). |

---

## 🔍 Technical Rationale (Architecture Choice)

* **Why React instead of Vanilla JS?** Plain JavaScript is too verbose for complex filtering logic. Using React prevents "reinventing the wheel" and ensures a scalable component-based architecture.
* **Desktop-First Approach:** Our core audience primarily uses second monitors while gaming. By focusing on Desktop, we can optimize the display of large data sets (match grids and filters) before considering a future mobile port.

---

## 📋 Phase 1: Brainstorming & Idea Evaluation

### Brainstorming Process
We conducted a collaborative research session using the **SCAMPER framework** and **Mind Mapping** to identify recurring issues in the gaming and e-sport industry. Our goal was to find a data-driven problem that fits a 4-week development cycle.

---

## 🎯 Project Refinement (The MVP)

### 1. Problem & Solution
* **The Problem**: E-sport fans currently struggle with fragmented data. They have to juggle between multiple wikis, social media, and streams to find match times.
* **The Solution**: **eSportCal**, a unified Desktop-First Web Application that aggregates professional schedules into a single, filtered interface.

### 2. Key Features (SMART Goals)
* **Unified Schedule**: Fetch and display 100% accurate match dates using the PandaScore API.
* **Dynamic Filtering**: Enable instant sorting by game (LoL, CS2, Valorant) using React state management.
* **User Accounts**: Provide secure login and profile customization using our custom Node.js backend and PostgreSQL database.

### 3. Project Scope
* **In-Scope**: API integration, filtering logic, SPA navigation, User Authentication (Login/Profile), and About/Contact pages.
* **Out-of-Scope**: Mobile-native application, real-time live score tickers, betting systems, or social chat features.

---

## ⚠️ Risk Management & Challenges

| Identified Risk | Mitigation Strategy |
| :--- | :--- |
| **Learning Curve** | Learning React/Vite from scratch in 4 weeks. | **Strategy**: Focus on functional components and intensive pair-programming. |
| **API Limitations** | No live scores on the free tier of PandaScore. | **Strategy**: Define the project strictly as a "Scheduling Tool" (Calendar). |
| **Time Management** | Broad scope (Login + Profile + API). | **Strategy**: Prioritize the core calendar functionality before secondary pages. |
| **Data Consistency** | Handling multiple timezones for global matches. | **Strategy**: Use libraries like `date-fns` to normalize time display to the user's local zone. |

---

## 📅 Progress Status
- [x] **Task 0**: Team Formation & Norms Definition.
- [x] **Task 1**: Research & Brainstorming.
- [x] **Task 2**: Decision & Project Refinement.
- [x] **Task 3**: Final Stage 1 Documentation.

---

# 📅 Stage 2: Project Planning - eSportCal

## 0. Project Charter Overview
The objective of this stage is to establish a clear roadmap for **eSportCal**. By outlining our timeline and milestones, we ensure that both team members (Antoine & Ilan) stay aligned on goals and deadlines throughout the development process.

---

## 1. High-Level Project Plan (Timeline)
We have mapped out the project over a 12-week period, ensuring enough time for technical research, development, and final testing.

| Phase | Duration | Focus | Key Deliverable |
| :--- | :--- | :--- | :--- |
| **Stage 1: Idea Development** | 1 week | Concept & Stack selection | [COMPLETED] Stage 1 Report |
| **Stage 2: Project Planning** | 1 week | Roadmap & Milestones | **[CURRENT]** Project Planning Doc |
| **Stage 3: Technical Doc** | 2 weeks | Architecture & Database design | System Diagrams & API Schema |
| **Stage 4: MVP Development** | 6 weeks | Coding the core features | Functional Web Application |
| **Stage 5: Project Closure** | 2 weeks | Deployment & Final Presentation | Live Demo & Project Review |

---

## 2. Detailed Milestone Breakdown

### Stage 3: Technical Documentation
* **Goal**: Create the "blueprint" of the application.
* **Key Tasks**: 
    * Design the Database Schema (PostgreSQL tables).
    * Map out the Backend API routes (Node.js/Express).
    * Create UI Mockups for the Dashboard and Login pages.

### Stage 4: MVP Development
* **Goal**: Build the functional version of eSportCal.
* **Development Sprints**:
    * **Sprint 1**: Backend & Database setup + PandaScore API connection.
    * **Sprint 2**: Frontend structure with React and main Calendar display.
    * **Sprint 3**: User Authentication (Login/Profile) and Data filtering.
    * **Sprint 4**: Bug fixing, UI polishing, and responsive adjustments.

### Stage 5: Project Closure
* **Goal**: Finalize and present the project.
* **Key Tasks**:
    * Final deployment of the application.
    * Creating the video demo and presentation slides.
    * Post-project reflection and documentation update.

---

## 3. Collaboration Strategy
* **Daily Sync**: A 10-minute "Stand-up" every morning at 10:00 AM to track progress and unblock any issues.
* **Task Management**: We use a shared **Trello Board** to move tasks from "To-Do" to "Done".
* **Peer Review**: Every piece of code is reviewed by both Antoine and Ilan before being merged into the main project.
