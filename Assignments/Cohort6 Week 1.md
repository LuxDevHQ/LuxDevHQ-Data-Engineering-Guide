### **Assignment: Server Access & Task Completion**

#### **Server Access Details**
Use the credentials below to access the assignment server:

- **Host:** 147.182.232.57
- **Username:** root
- **Password:** developer@123Abc

#### **Login Instructions**
Use SSH to access the server:

```bash
ssh root@147.182.232.57 -p 22
```
#### **Important Notes**
- Log in successfully to the server using the credentials provided.
- Confirm successful access before starting the assignment tasks.
- The assignment tasks are available in a GitHub repository.
- The same assignment will also be shared here and sent to your email for reference.

---

### **Intermediate Linux Practice Tasks**

### **Task 1: User & Group Management**
- Create a new group called `analytics_<yourname>`.
  *Example:*
  ```bash
  Username: analytics_Ganthony
  ```
- Create two users: `analyst1_<yourname>` and `analyst2_<yourname>`.
- Add both users to the `analytics_<yourname>` group.
- Ensure both users have valid home directories, and Verify group membership.

#### **Task 2: Controlled Directory Access**
- Create a directory called `/data/projects_<yourname>`.
- Set ownership as follows:
  - **Owner:** root
  - **Group:** analytics_<yourname>
- Configure permissions so that:
  - Members of `analytics_<yourname>` can read, write, and execute.
  - Others have no access.
- Ensure that new files and directories created inside inherit the group automatically.

---

#### **Task 3: File Permission Auditing**
- Inside `/data/projects_<yourname>`, create files and directories as different users.
- Identify files that are:
  - World-writable
  - Incorrectly owned
- Correct the permissions and ownership to follow best practices.

---

#### **Task 4: Sudo Configuration (Least Privilege)**
- Create a user called `support_<yourname>`.
- Grant the user limited sudo access to:
  - Restart services
  - View logs in `/var/log`
- Ensure the user does **not** have full root access.
- Test the sudo configuration by switching to the user.

---

#### **Task 5: Log Inspection & Troubleshooting**
- Navigate to `/var/log`.
- Identify system and authentication logs.
- View the last 50 log entries.
- Search for failed login attempts.
- Save your findings to `/home/support_<yourname>/log_report.txt`. 

Please reach out to 0796448232 for assistance if stuck. Do not use chat GPT (Stack overflow is allowed)


