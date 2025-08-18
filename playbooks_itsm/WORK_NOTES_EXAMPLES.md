# ServiceNow Work Notes and Activity Examples

## 🔧 **How to Add Work Notes and Comments**

The update incident playbook now supports adding work notes and activity entries through the `other:` section as per ServiceNow ITSM collection documentation.

## 📝 **Available Work Note Fields**

### 1. **work_notes** 
- Internal work notes visible to IT staff
- Used for technical updates and troubleshooting information

### 2. **comments**
- Customer-visible comments
- Used for communication with end users

### 3. **close_notes**
- Notes added when closing/resolving incidents
- Usually contains resolution details

### 4. **activity_due**
- Set due dates for follow-up activities

### 5. **additional_assignee_list**
- Add additional people to the incident

## 🚀 **Usage Examples**

### Example 1: Basic Work Notes
```yaml
# In your workflow job template survey or extra_vars:
work_notes: "Investigated the issue. Found root cause in database connection pool. Applied fix and monitoring."
```

### Example 2: Customer Communication
```yaml
# Use comments for customer-visible updates:
comments: "We have identified the issue and are working on a resolution. Expected completion: 2 hours."
```

### Example 3: Closing an Incident
```yaml
# When resolving/closing:
new_state: "6"  # Resolved
close_notes: "Issue resolved by restarting the application service. Monitoring for 24 hours to ensure stability."
close_code: "Solution provided"
```

### Example 4: Multiple Updates at Once
```yaml
# Multiple fields in one update:
new_state: "2"  # In Progress
work_notes: "Escalated to senior engineer. Currently investigating network connectivity issues."
new_assigned_to: "john.doe"
new_priority: "2"  # High priority
```

### Example 5: Adding Activity with Due Date
```yaml
work_notes: "Scheduled maintenance window for this weekend"
activity_due: "2024-01-15 18:00:00"
additional_assignee_list: "maintenance.team@company.com"
```

## 🤖 **Automatic Work Notes**

The playbook automatically adds work notes when:
- ✅ **auto_add_work_notes**: `true` (default)
- ✅ **No manual work notes**: You didn't provide `work_notes` or `comments`
- ✅ **Updates are made**: At least one field is being updated

**Automatic work note format:**
```
Incident updated via Ansible Automation Platform on 2024-01-10T14:30:00Z. 
Fields updated: state, priority, assigned_to. 
Updated by: ansible.service
```

### Disable Automatic Work Notes
```yaml
# Set in your job template extra_vars:
auto_add_work_notes: false
```

## 📋 **AAP Job Template Survey Configuration**

### Survey Questions for Work Notes:
```yaml
- variable: work_notes
  question: "Work Notes (Internal)"
  type: textarea
  required: false

- variable: comments  
  question: "Customer Comments (Visible)"
  type: textarea
  required: false

- variable: new_state
  question: "New Incident State"
  type: multiplechoice
  choices:
    - "1"  # New
    - "2"  # In Progress  
    - "3"  # On Hold
    - "6"  # Resolved
    - "7"  # Closed
  required: false

- variable: close_notes
  question: "Resolution Notes (if closing)"
  type: textarea
  required: false

- variable: close_code
  question: "Close Code (if resolving)"
  type: multiplechoice
  choices:
    - "Solution provided"
    - "Solved (Work Around)"
    - "Solved (Permanently)"
    - "Solved Remotely (Work Around)"
    - "Not Solved (Not Reproducible)"
    - "Closed/Resolved by Caller"
  required: false
```

## 🔍 **Work Notes in ServiceNow**

After running the update playbook, you'll see:

### Work Notes Tab:
- Internal technical updates
- Timestamp and user who made the update
- History of all work notes

### Activities Tab:
- Customer communications
- System-generated activities
- Due dates and assignments

### Activity Stream:
- Real-time feed of all incident activities
- Combination of work notes, comments, and field changes

## 💡 **Best Practices**

### 1. **Use Descriptive Work Notes**
```yaml
# ❌ Bad
work_notes: "Fixed it"

# ✅ Good  
work_notes: "Identified memory leak in application pool. Restarted IIS service and increased max memory allocation from 2GB to 4GB. Monitoring CPU and memory usage."
```

### 2. **Customer vs Internal Communication**
```yaml
# Internal technical details
work_notes: "Exception in auth.dll causing 500 errors. Applied hotfix KB123456."

# Customer-friendly update
comments: "We have resolved the login issue. Please try accessing the system again."
```

### 3. **State Changes with Context**
```yaml
# When changing state, explain why
new_state: "3"  # On Hold
work_notes: "Incident on hold pending vendor response. Ticket opened with Microsoft (Case #12345). Expected response within 24 hours."
```

### 4. **Resolution Documentation**
```yaml
# Complete resolution information
new_state: "6"  # Resolved
close_notes: "Root cause: Database connection timeout. Solution: Updated connection string timeout from 30s to 120s and optimized slow queries. Verified resolution in test environment."
close_code: "Solution provided"
```

## 🚨 **Troubleshooting Work Notes**

### Issue: Work notes not appearing
- **Check**: Work notes are in the `other:` section
- **Verify**: ServiceNow user has permission to add work notes
- **Debug**: Use the debug tasks to see what's being sent

### Issue: Customer can't see comments
- **Check**: Use `comments` field instead of `work_notes`
- **Verify**: Customer portal settings allow comment visibility

### Issue: Automatic work notes interfering
- **Solution**: Set `auto_add_work_notes: false`
- **Alternative**: Provide manual `work_notes` to override automatic ones
