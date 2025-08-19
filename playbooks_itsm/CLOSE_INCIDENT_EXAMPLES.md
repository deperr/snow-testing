# ServiceNow Incident Closure Playbook Examples

## 🎯 **Purpose**

The `snow_close_incident.yml` playbook is designed to properly close ServiceNow incidents with comprehensive resolution documentation. It can be used both **independently** and as **part of a workflow**.

## 🔧 **Key Features**

### ✅ **Workflow Integration**
- Automatically picks up incident identifiers from previous workflow steps
- Sets artifacts for downstream workflow activities
- Maintains variable passing consistency with create/update playbooks

### ✅ **Independent Operation**
- Can close incidents by providing incident number or sys_id directly
- Standalone operation for manual incident closure
- Full validation and error handling

### ✅ **Comprehensive Closure**
- Required closure documentation (close notes, close code)
- Optional field updates during closure
- Automatic work notes and closure tracking
- Proper state management (Resolved vs Closed)

## 🚀 **Usage Examples**

### **Example 1: Basic Independent Closure**
```yaml
# Close incident with minimal information
ansible-playbook snow_close_incident.yml -e '{
  "incident_number": "INC0012345",
  "close_notes": "Issue resolved by restarting the web service. Monitoring for stability.",
  "close_code": "Solution provided"
}'
```

### **Example 2: Comprehensive Closure with Updates**
```yaml
# Close incident with additional updates
ansible-playbook snow_close_incident.yml -e '{
  "incident_number": "INC0012345",
  "close_notes": "Root cause identified as memory leak in application. Applied patch v2.1.5 and increased memory allocation. Issue resolved and tested.",
  "close_code": "Solved (Permanently)",
  "resolution_method": "Patch Applied",
  "work_notes": "Applied permanent fix. System stable for 24 hours post-resolution.",
  "new_priority": "5",
  "new_assigned_to": "john.doe"
}'
```

### **Example 3: Workflow Integration**
```yaml
# In a workflow, automatically inherits incident details from previous steps
ansible-playbook snow_close_incident.yml -e '{
  "close_notes": "Automated resolution completed successfully. All systems operational.",
  "closure_method": "6"
}'
```

### **Example 4: Emergency Closure**
```yaml
# Quick closure with automatic notes
ansible-playbook snow_close_incident.yml -e '{
  "incident_number": "INC0012345",
  "close_code": "Closed/Resolved by Caller",
  "work_notes": "User confirmed issue resolved after network maintenance window."
}'
```

### **Example 5: Closure with State Selection**
```yaml
# Close as "Closed" instead of default "Resolved"
ansible-playbook snow_close_incident.yml -e '{
  "incident_sys_id": "abc123def456",
  "closure_method": "7",
  "close_notes": "Incident closed as duplicate of INC0011111.",
  "close_code": "Not Solved (Not Reproducible)"
}'
```

## 📋 **Available Variables**

### **🔑 Required Variables (one of):**
- `incident_number`: ServiceNow incident number (e.g., "INC0012345")
- `incident_sys_id`: ServiceNow sys_id of the incident

### **📝 Closure Variables:**
- `close_notes`: Resolution details and documentation
- `close_code`: Standardized closure reason
- `closure_method`: "6" (Resolved) or "7" (Closed) - default: "6"
- `resolution_method`: Method used to resolve the issue

### **🔄 Optional Update Variables:**
- `work_notes`: Additional work notes during closure
- `new_short_description`: Update incident title
- `new_description`: Update incident description  
- `new_priority`: Update priority (1-5)
- `new_assignment_group`: Update assignment group
- `new_assigned_to`: Update assigned user

### **⚙️ Control Variables:**
- `auto_add_closure_notes`: Auto-generate closure notes if not provided (default: true)

## 🎭 **Closure Codes Reference**

### **Standard ServiceNow Close Codes:**
- `"Solution provided"`: Issue resolved with a solution
- `"Solved (Work Around)"`: Temporary workaround provided
- `"Solved (Permanently)"`: Permanent fix implemented
- `"Solved Remotely (Work Around)"`: Remote temporary fix
- `"Not Solved (Not Reproducible)"`: Could not reproduce the issue
- `"Closed/Resolved by Caller"`: User confirmed resolution

### **Custom Close Codes:**
You can use custom close codes specific to your ServiceNow instance.

## 🔄 **Closure States**

### **State Values:**
- `"6"`: **Resolved** - Issue fixed, awaiting user confirmation
- `"7"`: **Closed** - Issue completely closed and archived
- `"8"`: **Canceled** - Not used by this playbook

### **When to Use Each:**
- **Resolved (6)**: Default - issue is fixed but may need user confirmation
- **Closed (7)**: Use when incident should be immediately closed and archived

## 🔗 **Workflow Integration Patterns**

### **Pattern 1: Create → Update → Close**
```yaml
# Workflow sequence
1. Create Incident (snow_create_incident.yml)
   └─ Sets: created_incident_number, created_incident_sys_id

2. Update Incident (snow_update_incident.yml)  
   └─ Uses: created_incident_* → Sets: updated_incident_*

3. Close Incident (snow_close_incident.yml)
   └─ Uses: updated_incident_* → Sets: closed_incident_*
```

### **Pattern 2: Create → Close**
```yaml
# Direct closure workflow
1. Create Incident (snow_create_incident.yml)
   └─ Sets: created_incident_number, created_incident_sys_id

2. Close Incident (snow_close_incident.yml)
   └─ Uses: created_incident_* → Sets: closed_incident_*
```

### **Pattern 3: Independent Closure**
```yaml
# Standalone closure
Close Incident (snow_close_incident.yml)
└─ Requires: incident_number OR incident_sys_id
└─ Sets: closed_incident_*
```

## 🛡️ **Validation & Error Handling**

### **Pre-Closure Validation:**
- ✅ Incident exists in ServiceNow
- ✅ Incident is not already closed/canceled
- ✅ Required closure information provided
- ✅ User has permissions to close incident

### **Error Scenarios:**
- **Incident Not Found**: Clear error message with troubleshooting steps
- **Already Closed**: Prevents double-closure with status information
- **Missing Close Info**: Requires either close_notes or close_code
- **Permission Denied**: ServiceNow permission error handling

## 📊 **Output & Artifacts**

### **Job Artifacts Set:**
```yaml
closed_incident_number: "INC0012345"
closed_incident_sys_id: "abc123def456"
closed_incident_state: "6"
closed_incident_close_code: "Solution provided"
closure_method: "6"
incident_url: "https://instance.service-now.com/nav_to.do?uri=incident.do?sys_id=abc123def456"
closure_timestamp: "2024-01-10T14:30:00Z"
closure_success: true
```

### **Console Output:**
```
✅ Incident closed successfully!
📋 Incident: INC0012345
🔄 Final State: Resolved
📝 Close Code: Solution provided
🔗 URL: https://instance.service-now.com/nav_to.do?uri=incident.do?sys_id=abc123def456
⏰ Closed: 2024-01-10T14:30:00Z
```

## 🎯 **AAP Job Template Configuration**

### **Survey Questions for Close Incident:**
```yaml
- variable: incident_number
  question: "Incident Number to Close"
  type: text
  required: true
  help_text: "ServiceNow incident number (e.g., INC0012345)"

- variable: close_notes
  question: "Resolution Notes"
  type: textarea
  required: true
  help_text: "Detailed explanation of how the incident was resolved"

- variable: close_code
  question: "Close Code"
  type: multiplechoice
  choices:
    - "Solution provided"
    - "Solved (Work Around)"
    - "Solved (Permanently)"
    - "Not Solved (Not Reproducible)"
    - "Closed/Resolved by Caller"
  default: "Solution provided"

- variable: closure_method
  question: "Closure State"
  type: multiplechoice
  choices:
    - "6"  # Resolved
    - "7"  # Closed
  default: "6"
  help_text: "6=Resolved (awaiting confirmation), 7=Closed (final)"

- variable: work_notes
  question: "Additional Work Notes"
  type: textarea
  required: false
  help_text: "Any additional notes for the incident record"
```

## 🔄 **Integration with Existing Workflow**

The close incident playbook is designed to seamlessly integrate with your existing `snow_create_incident.yml` and `snow_update_incident.yml` playbooks:

### **Variable Inheritance:**
- Automatically picks up `created_incident_*` variables
- Falls back to `updated_incident_*` variables  
- Uses manual `incident_*` variables as last resort

### **Artifact Compatibility:**
- Sets `closed_incident_*` artifacts for downstream use
- Maintains the same artifact structure pattern
- Compatible with existing workflow templates

This closure playbook completes your comprehensive ServiceNow ITSM automation suite! 🎉
