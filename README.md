# Cyber Club – Security Review (Post-Development)

## 1. Introduction
Cyber Club is a web application developed as part of my coop training to help cybersecurity students explore field-related courses and learning paths.
 Due to training scope and time constraints, the system was built with a development-first focus.
This document presents a post-development security review conducted to analyze potential security weaknesses and propose improvements from a cybersecurity perspective.

## 2. Scope of Review

### In Scope:

- Registration and rating forms

- Data visibility through application pages

- Input handling and validation logic

- Application-level design decisions

### Out of Scope:

- Production deployment

- Network or server hardening

- Real user data

- Authentication implementation (not originally included)


## 3. Assets

- Student registration data (user-provided personal info):
 Collected when a user fills the registration form: student ID, first name, last name, email, selected course, and whether they want notifications.

- Course rating data (user-submitted feedback):
 Collected through the rating page using multiple criteria. Ratings are displayed on the page using a repeater, and users can view previous ratings as soon as they open the page.

- Public exposure of student/rating information:
 Because there is no sign-in/privilege separation, student information and ratings are exposed publicly, which increases privacy/confidentiality risk.

- Quiz results, which determine whether a user is more suited for Red Team or Blue Team.
While not sensitive, this data must remain correct for the feature to function as intended.

## 4. Entry points
Based on user interaction with the Cyber Club application, the following entry points were identified:

### Registration Page:


- Users can enter personal information such as student ID, name, email, selected course, and notification preference.


- Users can submit this information to insert a new registration record.


- Users can update existing registration data, including changing the selected course or notification option.


- Users can delete registration records using the delete action.


- The same registration data is displayed back to users through a GridView.


### Rating Page:


- Users can submit course ratings based on multiple criteria.


- Previously submitted ratings are displayed to users using a repeater, even without submitting a new rating.


### Contact Us Page:


- Users can enter and submit free-form messages that are sent directly to the developer.


### Export to Excel (Evaluation Feature):


- Users can export registration data to an Excel file.


- The exported data is the same information already displayed in the GridView and was included for evaluation purposes.


- In a real-world deployment, this functionality would be restricted to administrative users.
 

Each form submission or user-triggered action that inserts, updates, deletes, displays, or exports data is considered an entry point from a security perspective.


## 5. Threats

The following threats were identified based on how users can interact with the Cyber Club application and how data is handled:

- Unauthorized data modification by normal users
 Due to the absence of authentication and access control, a normal user could update or delete other users’ registration data.
 The same lack of control allows users to submit false or misleading ratings, even without malicious intent.

- Submission of fake or misleading data
 Users can register using false information and submit ratings that do not reflect real experience, affecting the integrity and reliability of the system.

- Unauthorized access to user data
 Because there is no privilege separation, users can view student information and ratings that they should not have access to.

- Social engineering attacks using exposed information
 Registration data could be abused to contact users and impersonate Cyber Club or course mentors.
 An attacker could send spam, request payments, or distribute malicious links by exploiting user trust.

- Malicious use of the Contact Us page
 A malicious user could send harmful or deceptive messages directly to the developer.
 If opened without caution, such messages or attachments could lead to malware infection.

- Abuse without technical exploitation
 The system can be misused easily without advanced hacking techniques.
 Most attacks rely on abusing exposed information and open functionality rather than exploiting technical vulnerabilities.

- Design-related exposure due to evaluation requirements
 Certain features (such as public GridView display and Export to Excel) were added for evaluation purposes rather than real-world usage.
 These design choices increase exposure and make data easily accessible, lowering the effort required for misuse.

## 6. Findings

### Finding 1: Lack of Authentication and Access Control

**Observation:**
 The application does not implement authentication or authorization.
 Any user can access the system and view available information without logging in, and there is no privilege separation to restrict access to sensitive data.

**Type:**
 Design issue

**Why This Matters?**
 Without authentication and access control, the system cannot protect user information.
 This primarily affects confidentiality, as data is visible to anyone regardless of identity or role.

**Potential Impact:**

- Exposed information can be misused for social engineering attacks, where attackers contact users and impersonate Cyber Club or trusted parties.

**Risk Level:**
 High

**Public Exposure Consideration:**
 This issue would be more severe if the application were publicly accessible, as modern applications rely on authentication to protect user data.

**Recommendation:**
Introduce authentication and role-based access control to restrict access to user data and sensitive actions. Separate regular users from administrative users.

### Finding 2: Unrestricted Visibility and Modification of User Data

**Observation:**
 User information is displayed publicly, and data-modifying actions are available without access restrictions.
 Any user can view, update, or delete other users’ data because there is no privilege or role control.

**Type:**
 Design issue

**Why This Matters?**
 Allowing unrestricted access to user data violates basic security principles and exposes information to misuse.
 It also removes accountability, as actions cannot be tied to a specific authorized user.

**Potential Impact:**

- Attackers could collect user information and use it for social engineering, spam, or malware distribution.

**Risk Level**
 High

**Public Exposure Consideration:**
 If the application were publicly available, the lack of restrictions would increase the number of potential victims and make abuse easier.

**Recommendation:**
Limit data visibility and modification actions based on user roles, and restrict update/delete functionality to authorized users only

### Finding 3: Risk of Malicious Content via Contact Us Page

**Observation:**
 The Contact Us page allows any user to submit messages that are sent directly to the developer.
 There is a possibility that an attacker could send a message that appears legitimate but contains malicious content.

**Type:**
 Implementation issue

**Why This Matters?**
 Messages submitted through this feature are delivered directly to the developer.
 If a malicious message is trusted and opened, it could negatively affect the security of the developer.

**Potential Impact:**

- Damage to the developer’s device

- Possible compromise of sensitive data stored on the device

- Risk is not limited to script-based attacks and could include other malicious content


**Risk Level:**
 Medium

**Public Exposure Consideration:**
 This risk would become more relevant if the application became more widely used, as increased visibility could attract malicious submissions.

**Recommendation:**
Apply input validation and filtering on Contact Us submissions and introduce safeguards such as message review or rate limiting before delivering messages to the developer.

### Finding 4: Exposure of User Identity Through Public Course Ratings

**Observation:**
 Course ratings are displayed publicly and include the name entered by the user.
 Although users can choose to enter only their first name, this still exposes identifiable information along with the course they took.

**Type:**
 Design issue

**Why This Matters?**
 Displaying identifiable user information alongside course activity can make individuals easier to target.
 Even limited information, such as a name and course association, can be sufficient for attackers to build trust during social engineering attempts.

**Potential Impact:**

- Attackers could look up users on social media using their name

- Attackers could impersonate Cyber Club staff and reference the course the user rated

- Users could be tricked into sharing additional personal information or clicking malicious links


**Risk Level:**
 Medium

**Public Exposure Consideration:**
 If the application were publicly accessible or became more widely used, the number of exposed users would increase, making social engineering attacks more likely.

**Recommendation:**
Avoid displaying identifiable user information publicly, or anonymize ratings to prevent user identification.

### Finding 5: User-Provided Data Rendered Without Output Encoding

**Observation:**
 User-provided data is rendered back to application pages using Labels, GridView, and Repeater controls.
 No output encoding, filtering, or transformation is applied before displaying this data to users.

**Type:**
 Implementation issue

**Why This Matters?**
Rendering user-provided input without encoding means the application relies on users submitting safe and well-formed content.
If unexpected input is submitted, the browser may interpret it in unintended ways, which creates a security risk.

**Potential Impact:**

- Unintended execution of content in the browser

- Display manipulation or unexpected page behavior

- Increased risk when combined with publicly visible data and lack of access control

**Risk Level:**
Medium

**Public Exposure Consideration:**
If the application were publicly accessible, the likelihood and impact of misuse would increase, as more users could submit unexpected input.

**Recommendation:**
 Apply output encoding when rendering user-provided data to ensure it is displayed as plain text and not interpreted by the browser.

## 7. Conclusion

This security review showed that while the Cyber Club application is strong from a functionality perspective and includes some basic security considerations, it would require significantly more security controls to be suitable for public use. The application works well in terms of features, but its security is limited due to the absence of essential protections such as access control and data protection mechanisms.
Through conducting this review, it became clear that building an application is much easier than securing it. This assessment helped highlight the importance of protecting user data and privacy, which emerged as the primary area of risk. It also reinforced the reality that security is often overlooked during development, even though it is critical for long-term success.
If this application were deployed in a real-world environment, the impact of these security gaps would increase as the number of users grows. A larger user base would mean greater responsibility, as security weaknesses could lead to user harm and loss of trust in the platform. This review emphasizes the need to treat security as an essential part of application design rather than an afterthought.

## 8. Disclaimer
This review was conducted for educational purposes only.
 No real users, patient data, or organizational systems were accessed.
