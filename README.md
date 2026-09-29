# Applied_Devops

#Hospital Management -Devops Project

## 1. Project overview

The Hospital Management System is a web application for managing patient information, doctors, appointments, and basic medical records. Receptionists register patients and arrange appointments, administrators manage doctors and system users, and doctors review their assigned appointments and record consultation outcomes. Authentication and role-based access control (RBAC) protect sensitive information.

**The deliverable is both the working application and the DevOps pipeline.** The project will demonstrate how changes are developed, tested, built, delivered, deployed, verified, and monitored. The implementation will be delivered incrementally through a prioritized product backlog.

## 2. Requirement analysis

### 2.1 Functional requirements

Requirements are grouped by the six main areas of the project scenario. Each identifier can be linked to backlog stories and tests.

#### Patient management

- **FR-01:** A receptionist shall be able to register a patient with name, date of birth, contact information, and address.
- **FR-02:** Authorized staff shall be able to search for and view patient basic information.
- **FR-03:** A receptionist shall be able to update patient basic and contact information.
- **FR-04:** A receptionist shall be able to deactivate a patient without deleting historical appointments or medical records.
- **FR-05:** A deactivated patient shall not be selectable for a new appointment.

#### Doctor management

- **FR-06:** An administrator shall be able to register a doctor with name, contact details, and medical specialization.
- **FR-07:** An administrator shall be able to view and update doctor information.
- **FR-08:** An administrator shall be able to deactivate a doctor without removing historical appointments or medical records.
- **FR-09:** Authorized staff shall be able to view doctors' specializations and whether they are active and available for new appointments.

#### Appointment management

- **FR-10:** A receptionist shall be able to create an appointment by selecting an active patient and an active doctor and entering its date, time, and reason for visit.
- **FR-11:** An appointment shall have one of three statuses: Scheduled, Completed, or Cancelled.
- **FR-12:** The system shall reject an appointment that would give the same doctor two non-cancelled appointments at the same date and time, including when an appointment is rescheduled.
- **FR-13:** Authorized staff shall be able to view appointments within the scope of their role.
- **FR-14:** A receptionist shall be able to change an appointment's date, time, doctor, or reason and cancel an appointment.
- **FR-15:** A doctor shall be able to mark an assigned appointment as Completed after the visit.

#### Medical records

- **FR-16:** A doctor shall be able to view basic information about patients with appointments assigned to that doctor.
- **FR-17:** After an assigned appointment, a doctor shall be able to create a medical visit record containing the visit date, diagnosis, and consultation notes, linked to that patient and appointment.
- **FR-18:** An authorized doctor shall be able to view previous medical records for a patient with whom they have an assigned appointment.
- **FR-19:** Medical visit records shall be retained when a patient or doctor is deactivated.
- **FR-20:** Receptionists shall not be able to create, modify, or view diagnoses or consultation notes.

#### Authentication and role-based access

- **FR-21:** Users shall log in before accessing protected application functionality.
- **FR-22:** The system shall distinguish Administrator, Receptionist, and Doctor roles and enforce permissions for each role.
- **FR-23:** An administrator shall be able to create, view, update, and deactivate user accounts and assign roles.
- **FR-24:** A doctor shall be able to see only appointments assigned to that doctor and medical information authorized by those assignments.
- **FR-25:** Unauthenticated or unauthorized requests shall be denied, including requests made directly to application endpoints.

#### CSV reporting

- **FR-26:** A doctor shall be able to export authorized appointment and medical-visit data as a CSV report.
- **FR-27:** The export shall not include patients or appointments outside the doctor's authorized scope.

### 2.2 Non-functional requirements

The first four requirements capture the scenario's explicit quality constraints; the remaining ones support a secure, testable DevOps delivery process.

- **NFR-01 — Performance:** Under the expected demonstration workload, normal application requests should normally complete within **2 seconds**. Verification: measure response times for representative login, search, detail, and appointment requests during the demonstration.
- **NFR-02 — Availability:** The application shall remain available during normal operation. Verification: its health endpoint remains reachable during the demonstration except while a deliberately simulated failure is being recovered.
- **NFR-03 — Recovery:** The deployment environment shall automatically restart an application or container that fails. Verification: stop or terminate the application container and confirm that it returns to a healthy state without manual restart.
- **NFR-04 — Security:** Passwords shall never be stored in plain text; protected functions shall require authentication and shall enforce role permissions. Verification: inspect stored password representations and test unauthenticated and unauthorized requests.
- **NFR-05 — Data preservation:** Application container restarts shall not erase persistent patient, doctor, appointment, or medical-record data. Verification: create a record, restart the application container, and confirm the record remains.
- **NFR-06 — Secret handling:** Database credentials, signing secrets, and other sensitive configuration values shall not be committed to the public repository. They shall be supplied through deployment configuration or environment variables.
- **NFR-07 — Usability:** The interface shall support common desktop and mobile screen sizes and display clear validation errors for missing or invalid input.
- **NFR-08 — Automated testing:** Automated tests shall cover core rules, especially appointment conflict prevention and authorization boundaries.
- **NFR-09 — Continuous integration:** On every push and pull request to the main development branch, CI shall build the application and run automated tests; failing checks shall be visible.
- **NFR-10 — Delivery and deployment:** The application shall be buildable as a container image and deployable through a documented, repeatable process, with automation in the pipeline where the deployment environment permits it.
- **NFR-11 — Verification and monitoring:** The deployed application shall expose a health check and provide accessible logs so that deployment health, failures, and recovery can be checked.
- **NFR-12 — Maintainability:** The repository shall contain build, test, run, and deployment instructions; changes shall be tracked through issues and pull requests.

## 3. Roles and access control

### Role descriptions

- **Administrator:** Manages doctor profiles, doctor active status, and system user accounts and roles. Does not receive automatic access to medical notes.
- **Receptionist:** Manages patient basic information and appointments. Cannot view or edit diagnoses and consultation notes.
- **Doctor:** Views assigned appointments and the basic/medical information of patients with assigned appointments, records outcomes for own appointments, and exports only authorized data.

### RBAC matrix

“Assigned patients” means patients with at least one appointment assigned to the logged-in doctor. “Own appointments” means appointments assigned to that doctor. All permissions require login.

| Action | Administrator | Receptionist | Doctor |
|---|---|---|---|
| Manage users and roles | Yes | No | No |
| Create, update, or deactivate doctors | Yes | No | No |
| View doctor details and specialization | Yes | Yes | Yes |
| Register or update patient basic information | No | Yes | No |
| Deactivate patient | No | Yes | No |
| Search/view patient basic information | No | Yes | Assigned patients only |
| Create or reschedule appointment | No | Yes | No |
| View appointments | No | All | Own appointments only |
| Cancel appointment | No | Yes | No |
| Mark appointment Completed | No | No | Own appointments only |
| View diagnoses and consultation notes | No | No | Assigned patients only |
| Create medical visit record | No | No | Own completed appointments only |
| Edit medical visit record | No | No | No (out of initial scope) |
| Export CSV report | No | No | Authorized data only |

### Decisions where the scenario is ambiguous

- The administrator manages users and doctors but does not see medical notes by default. Managing accounts is different from providing care; denying clinical access limits exposure of sensitive data.
- Receptionists can see patient demographics needed for registration and scheduling but not diagnoses or consultation notes.
- Doctors can view previous records for an assigned patient to support care, but cannot browse unrelated patients. A doctor can record a visit only for an appointment assigned to them and marked Completed.
- Receptionists cancel appointments. Doctors can complete their own appointments but cannot reschedule or cancel them; scheduling changes remain with receptionists.
- CSV exports are limited to the logged-in doctor's authorized patients, appointments, and medical visit records. Administrators and receptionists have no CSV export permission in this project scope.
- Deactivation is a soft delete: historical links stay intact, while inactive patients and doctors cannot be used for new appointments. A cancelled appointment does not occupy a doctor's time slot.

## 4. Product backlog

The stories below are ordered by implementation priority. Acceptance criteria are specified for the highest-priority stories; later stories will be refined before they enter a sprint.

| Order | ID | User story | Priority | Acceptance criteria / planned result |
|---:|---|---|---|---|
| 1 | US-01 | As a user, I want to log in so that only authenticated users enter the application. | High | Valid credentials start an authenticated session; invalid credentials do not; protected endpoints reject unauthenticated requests. |
| 2 | US-02 | As an administrator, I want to manage user accounts and roles so that staff receive the correct access. | High | Administrator can create, update, and deactivate accounts and assign one of the three roles; other roles cannot. |
| 3 | US-03 | As a receptionist, I want to register a patient so that their basic details are recorded. | High | Name, date of birth, contact details, and address are saved; missing required fields show errors. |
| 4 | US-04 | As a receptionist, I want to search for and view patients so that I can find the correct record. | High | Search returns matching patients and selecting a result shows basic details; diagnoses are not shown. |
| 5 | US-05 | As a receptionist, I want to update patient details so that changes remain accurate. | High | Edited contact details and address persist and appear on the patient detail page. |
| 6 | US-06 | As an administrator, I want to register doctors so that appointments can be assigned to them. | High | A doctor with name, contact details, and specialization can be saved and appears in the active doctor list. |
| 7 | US-07 | As a receptionist, I want to schedule appointments so that patients can see doctors. | High | An appointment stores an active patient, active doctor, date, time, reason, and Scheduled status. |
| 8 | US-08 | As a receptionist, I want the system to reject doctor double-bookings so that a doctor is not scheduled twice. | High | Creating or rescheduling a non-cancelled appointment into an occupied doctor/date/time slot is rejected. |
| 9 | US-09 | As a doctor, I want to see only my appointments so that I can prepare for my visits. | High | The doctor sees assigned appointments; requests for another doctor's appointments are denied. |
| 10 | US-10 | As a doctor, I want to record a completed visit so that its diagnosis and notes are retained. | High | A visit record has date, diagnosis, notes, patient, and own completed appointment; a receptionist cannot create one. |
| 11 | US-11 | As a doctor, I want to see an assigned patient's previous records so that I understand their history. | High | Prior visit records for assigned patients are visible; unrelated patients' records are denied. |
| 12 | US-12 | As a receptionist, I want to update or cancel appointments so that schedule changes are reflected. | High | A changed appointment persists and is checked for conflicts; cancelled appointments show Cancelled status. |
| 13 | US-13 | As a receptionist, I want to deactivate patients without losing history so that old appointments remain traceable. | Medium | Inactive patient is not selectable for new appointments; historical links and records remain. |
| 14 | US-14 | As an administrator, I want to update and deactivate doctors so that the available-doctor list is accurate. | Medium | Updated specialization is displayed; inactive doctor cannot be booked; historical links remain. |
| 15 | US-15 | As a doctor, I want to export authorized data as CSV so that I can generate a report. | Medium | CSV downloads successfully and excludes unrelated patients and appointments. |
| 16 | US-16 | As a developer, I want automated builds and tests in CI so that changes are checked before delivery. | High | Pushes and pull requests run the build and automated tests; failures are visible. |
| 17 | US-17 | As a developer, I want a containerized deployment and automatic restart so that the service can recover from failure. | High | Application starts from documented container configuration and recovers after its container is terminated. |
| 18 | US-18 | As an operator, I want health checks and logs so that I can verify deployments and investigate failures. | Medium | Health endpoint and logs can be checked after deployment; failure and recovery can be observed. |

### Definition of Done

A story is Done when its acceptance criteria are met; authorization and validation have been checked where relevant; appropriate automated tests pass; CI builds and tests successfully; the change has been reviewed and merged; and documentation or deployment instructions have been updated when affected. A deployable change must pass a health check after deployment.

## 5. Process and ceremonies

### Sprint length and outline

The project uses **one-week sprints**. At the start of a sprint, the highest-priority ready stories are selected. During the sprint, each story moves from GitHub Issue to implementation branch, automated tests, pull request, CI checks, review, and merge. The resulting increment is built and deployed, then verified using health checks and logs. At the end of the sprint, completed work is demonstrated and the process is improved.

An initial sprint outline is:

| Sprint | Main focus | Expected demonstrable outcome |
|---|---|---|
| 1 | Repository, requirements, basic application structure, database, authentication, and CI baseline | Application starts; login and automated build/test pipeline can be demonstrated. |
| 2 | Patient and doctor management with RBAC | Receptionist manages patients; administrator manages doctors and users. |
| 3 | Appointments, conflict prevention, and medical records | Appointments can be managed; doctors can record visits for authorized patients. |
| 4 | CSV export, additional tests, container deployment, recovery, verification, and monitoring | End-to-end demo plus pipeline, recovery, and monitoring evidence. |

The outline is a plan, not a claim that these features have already been implemented. Scope can be adjusted during refinement based on progress.

### Ceremonies

| Ceremony | When | Purpose |
|---|---|---|
| Sprint Planning | Start of each one-week sprint | Select ready stories, agree on a sprint goal, and identify implementation tasks. |
| Daily Stand-up | Each working day | Briefly discuss completed work, next work, and blockers. For a solo project, use a short written progress note. |
| Sprint Review and Retrospective | End of each sprint | Demonstrate completed work, check acceptance criteria, and record one or two process improvements. |

### Backlog refinement triggers

Refinement takes place before each Sprint Planning session and whenever a story is unclear, too large for one sprint, lacks testable acceptance criteria, depends on unresolved technical work, or a new issue changes the priority order. Before a story is selected for a sprint, it should have a clear role and outcome, acceptance criteria, and a manageable scope.
