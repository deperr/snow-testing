# Complete ServiceNow ITSM Workflow Guide

## 🎯 **Overview**

This guide shows how to use the complete ServiceNow ITSM automation suite in Ansible Automation Platform workflows. The suite includes three playbooks that work together seamlessly:

1. **`snow_create_incident.yml`** - Creates new incidents
2. **`snow_update_incident.yml`** - Updates existing incidents  
3. **`snow_close_incident.yml`** - Closes incidents with proper resolution

## 🔄 **Complete Workflow Patterns**

### **Pattern 1: Full Lifecycle Workflow**
```
Create Incident → Update Incident → Close Incident
      ↓                ↓               ↓
  created_*        updated_*       closed_*
```

### **Pattern 2: Create and Close**
```
Create Incident → Close Incident
      ↓               ↓
  created_*       closed_*
```

### **Pattern 3: Update and Close**
```
Update Incident → Close Incident
      ↓               ↓
  updated_*       closed_*
```

## 📋 **Variable Flow Between Playbooks**

### **Create → Update → Close**
```yaml
# Create Incident sets:
created_incident_number: "INC0012345"
created_incident_sys_id: "abc123def456"

# Update Incident uses and sets:
target_incident_number: "{{ created_incident_number }}"  # Input
updated_incident_number: "INC0012345"                    # Output
updated_incident_sys_id: "abc123def456"                  # Output

# Close Incident uses and sets:
target_incident_number: "{{ updated_incident_number }}"  # Input
closed_incident_number: "INC0012345"                     # Output
closed_incident_sys_id: "abc123def456"                   # Output
```

## 🚀 **AAP Workflow Template Setup**

### **Step 1: Create Individual Job Templates**

#### **Job Template: "ServiceNow - Create Incident"**
```yaml
Name: ServiceNow - Create Incident
Playbook: playbooks_itsm/snow_create_incident.yml
Survey Enabled: Yes
Ask Variables on Launch: Yes
```

**Survey Questions:**
```yaml
- variable: incident_short_description
  question: "Incident Title"
  type: text
  required: true
  min: 10
  max: 160

- variable: incident_description
  question: "Incident Description"
  type: textarea
  required: false

- variable: incident_urgency
  question: "Urgency"
  type: multiplechoice
  choices: ["high", "medium", "low"]
  default: "medium"

- variable: incident_impact
  question: "Impact"
  type: multiplechoice
  choices: ["high", "medium", "low"]
  default: "medium"
```

#### **Job Template: "ServiceNow - Update Incident"**
```yaml
Name: ServiceNow - Update Incident
Playbook: playbooks_itsm/snow_update_incident.yml
Survey Enabled: Yes (optional)
Ask Variables on Launch: Yes
```

**Survey Questions (Optional):**
```yaml
- variable: new_state
  question: "New State"
  type: multiplechoice
  choices: ["2", "3", "6"]  # In Progress, On Hold, Resolved
  required: false

- variable: work_notes
  question: "Work Notes"
  type: textarea
  required: false

- variable: new_assignment_group
  question: "Assign to Group"
  type: text
  required: false
```

#### **Job Template: "ServiceNow - Close Incident"**
```yaml
Name: ServiceNow - Close Incident
Playbook: playbooks_itsm/snow_close_incident.yml
Survey Enabled: Yes (optional)
Ask Variables on Launch: Yes
```

**Survey Questions (Optional):**
```yaml
- variable: close_notes
  question: "Resolution Notes"
  type: textarea
  required: false
  help_text: "How was this incident resolved? (optional - auto-generated if empty)"

- variable: closure_method
  question: "Closure State"
  type: multiplechoice
  choices: ["6", "7"]  # Resolved, Closed
  default: "6"
  help_text: "6=Resolved (awaiting confirmation), 7=Closed (final)"

- variable: close_code
  question: "Close Code"
  type: multiplechoice
  choices:
    - "Solution provided"
    - "Solved (Permanently)"
    - "Solved (Work Around)"
    - "Not Solved (Not Reproducible)"
    - "Closed/Resolved by Caller"
  default: "Solution provided"
```

### **Step 2: Create Workflow Template**

#### **Workflow: "ServiceNow - Complete Incident Management"**
```yaml
Name: ServiceNow - Complete Incident Management
Organization: Default
Survey Enabled: Yes
Ask Variables on Launch: Yes
```

**Workflow Survey:**
```yaml
- variable: incident_short_description
  question: "What is the issue?"
  type: text
  required: true
  min: 10
  max: 160

- variable: incident_description
  question: "Detailed Description"
  type: textarea
  required: false

- variable: incident_urgency
  question: "How urgent is this?"
  type: multiplechoice
  choices: ["high", "medium", "low"]
  default: "medium"

- variable: auto_close
  question: "Automatically close after creation?"
  type: multiplechoice
  choices: ["yes", "no"]
  default: "yes"

- variable: resolution_notes
  question: "Resolution Notes (if auto-closing)"
  type: textarea
  required: false
  default: "Issue resolved through automated workflow process."
```

### **Step 3: Configure Workflow Nodes**

#### **Node 1: Create Incident**
```yaml
Node Type: Job Template
Job Template: ServiceNow - Create Incident
Identifier: create_incident
```

#### **Node 2: Update Incident (Optional)**
```yaml
Node Type: Job Template
Job Template: ServiceNow - Update Incident
Identifier: update_incident
Parent: create_incident (on success)
```

#### **Node 3: Close Incident**
```yaml
Node Type: Job Template
Job Template: ServiceNow - Close Incident
Identifier: close_incident
Parent: update_incident (on success) OR create_incident (on success)
Extra Variables:
  close_notes: "{{ resolution_notes }}"
  closure_method: "6"
```

## 🎭 **Workflow Examples**

### **Example 1: Simple Create and Auto-Close**
```yaml
# Workflow Input:
incident_short_description: "Email server not responding"
incident_urgency: "high"
auto_close: "yes"
resolution_notes: "Email server restarted successfully. Service restored."

# Workflow Flow:
1. Create Incident → INC0012345 created
2. Close Incident → INC0012345 closed as resolved
```

### **Example 2: Full Lifecycle with Updates**
```yaml
# Workflow Input:
incident_short_description: "Database performance degradation"
incident_urgency: "medium"
auto_close: "no"

# Workflow Flow:
1. Create Incident → INC0012346 created
2. Update Incident → Set to "In Progress", add work notes
3. Close Incident → Manual closure with detailed resolution
```

### **Example 3: Emergency Closure**
```yaml
# Workflow Input:
incident_short_description: "Network outage - Site A"
incident_urgency: "high"
auto_close: "yes"
resolution_notes: "Network issue resolved. Faulty router replaced."

# Workflow with immediate closure:
1. Create Incident → INC0012347 created
2. Close Incident → Immediately closed as resolved
```

## 📊 **Workflow Output Tracking**

### **Artifacts Available After Workflow:**
```yaml
# From Create Step:
created_incident_number: "INC0012345"
created_incident_sys_id: "abc123def456"
incident_url: "https://instance.service-now.com/..."

# From Update Step (if used):
updated_incident_number: "INC0012345"
updated_incident_state: "2"
fields_updated: ["state", "work_notes", "assignment_group"]

# From Close Step:
closed_incident_number: "INC0012345"
closed_incident_state: "6"
closed_incident_close_code: "Solution provided"
closure_success: true
closure_timestamp: "2024-01-10T14:30:00Z"
```

## 🔧 **Advanced Workflow Configurations**

### **Conditional Closure Based on Survey**
```yaml
# In workflow node conditions:
Close Incident Node:
  Parent: create_incident
  Run Condition: "{{ auto_close == 'yes' }}"
  
Manual Update Node:
  Parent: create_incident  
  Run Condition: "{{ auto_close == 'no' }}"
```

### **Parallel Workflow Branches**
```yaml
# Create incident then branch based on priority
Create Incident → High Priority Path (immediate assignment)
              → Normal Priority Path (standard process)
              → Low Priority Path (scheduled resolution)
```

### **Error Handling Workflows**
```yaml
# Add approval nodes and failure handling
Create Incident → Approval Node → Update Incident → Close Incident
              → Failure Node → Notify Support Team
```

## 🎯 **Best Practices**

### **1. Survey Design**
- **Keep surveys minimal** for workflow efficiency
- **Use conditional surveys** based on workflow path
- **Provide sensible defaults** to reduce user input

### **2. Variable Management**
- **Let workflows handle variable passing** automatically
- **Use survey variables only for user-specific input**
- **Avoid overriding workflow artifacts** unless necessary

### **3. Error Handling**
- **Add failure notifications** for critical workflows
- **Include rollback procedures** for complex operations
- **Monitor workflow success rates** and optimize accordingly

### **4. Testing**
- **Test complete workflows** end-to-end in development
- **Verify ServiceNow integration** with test instances
- **Validate variable passing** between all workflow steps

This complete workflow setup provides end-to-end ServiceNow incident management automation with full lifecycle tracking and professional closure handling! 🎉
