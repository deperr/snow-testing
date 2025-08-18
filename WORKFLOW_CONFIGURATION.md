# AAP Workflow Configuration Guide

## Problem Analysis: Why Variables Weren't Passing

### Root Causes Identified:

1. **❌ Incorrect Module Defaults Structure**
   - **Wrong**: Individual module targets (`servicenow.itsm.incident:`, `servicenow.itsm.incident_info:`)
   - **Correct**: Group target (`group/servicenow.itsm.servicenow:`)

2. **❌ Variable Scope Issues**
   - `set_fact` variables don't automatically transfer between job templates
   - Need to use `set_stats` with `aggregate: true` for workflow variable passing

3. **❌ Incorrect Artifact Configuration**
   - `aggregate: false` prevents variables from being available to subsequent jobs
   - Variable names in artifacts must match exactly what the next job expects

## ✅ Fixed Configuration

### Create Incident Playbook (`snow_create_incident.yml`)
```yaml
# Fixed module_defaults structure
module_defaults:
  group/servicenow.itsm.servicenow:
    instance: "{{ servicenow_instance }}"
    username: "{{ servicenow_username }}"
    password: "{{ servicenow_password }}"
    validate_certs: true

# Fixed artifact configuration
- name: Save incident details to job artifacts
  ansible.builtin.set_stats:
    data:
      created_incident_number: "{{ created_incident.record.number }}"
      created_incident_sys_id: "{{ created_incident.record.sys_id }}"
      created_incident_state: "{{ created_incident.record.state }}"
      created_incident_priority: "{{ created_incident.record.priority }}"
      incident_url: "https://{{ servicenow_instance }}/nav_to.do?uri=incident.do?sys_id={{ created_incident.record.sys_id }}"
    per_host: false
    aggregate: true  # ✅ CRITICAL: Must be true for workflow sharing
```

### Update Incident Playbook (`snow_update_incident.yml`)
```yaml
vars:
  # ✅ Correct variable retrieval from workflow artifacts
  target_incident_number: "{{ created_incident_number | default(incident_number | default('')) }}"
  target_incident_sys_id: "{{ created_incident_sys_id | default(incident_sys_id | default('')) }}"
```

## AAP Workflow Template Configuration

### Step 1: Create Job Templates

#### Job Template 1: "Create ServiceNow Incident"
- **Playbook**: `playbooks_itsm/snow_create_incident.yml`
- **Credentials**: ServiceNow credential
- **Survey Variables** (optional):
  ```yaml
  - variable: incident_short_description
    question: "Incident Title"
    type: text
    required: true
  
  - variable: incident_urgency
    question: "Urgency"
    type: multiplechoice
    choices: ["high", "medium", "low"]
    default: "medium"
  ```

#### Job Template 2: "Update ServiceNow Incident"
- **Playbook**: `playbooks_itsm/snow_update_incident.yml`
- **Credentials**: ServiceNow credential
- **Survey Variables** (optional):
  ```yaml
  - variable: new_state
    question: "New State"
    type: multiplechoice
    choices: ["2", "6", "7"]  # In Progress, Resolved, Closed
    
  - variable: work_notes
    question: "Work Notes"
    type: textarea
  ```

### Step 2: Create Workflow Template

1. **Create Workflow Template**: "ServiceNow Incident Management"
2. **Add Workflow Nodes**:
   - **Node 1**: "Create ServiceNow Incident" (job template)
   - **Node 2**: "Update ServiceNow Incident" (job template)
   - **Connection**: Node 1 → Node 2 (On Success)

### Step 3: Variable Flow Configuration

**No additional configuration needed!** The variables flow automatically:

```
Job 1 (Create) → set_stats → Job 2 (Update)
created_incident_number → target_incident_number
created_incident_sys_id → target_incident_sys_id
```

## ServiceNow Credential Configuration

### Create Custom Credential Type in AAP:

1. **Name**: ServiceNow
2. **Input Configuration**:
```yaml
fields:
  - id: servicenow_instance
    type: string
    label: ServiceNow Instance
    help_text: "Your ServiceNow instance (e.g., dev12345.service-now.com)"
  - id: servicenow_username
    type: string
    label: Username
  - id: servicenow_password
    type: string
    label: Password
    secret: true
required:
  - servicenow_instance
  - servicenow_username
  - servicenow_password
```

3. **Injector Configuration**:
```yaml
extra_vars:
  servicenow_instance: "{{ servicenow_instance }}"
  servicenow_username: "{{ servicenow_username }}"
  servicenow_password: "{{ servicenow_password }}"
```

## Testing the Workflow

### Manual Testing Steps:

1. **Run Create Job** with test data:
   - `incident_short_description`: "Test Incident"
   - `incident_urgency`: "medium"

2. **Check Job Artifacts** in AAP UI:
   - Should see `created_incident_number` and `created_incident_sys_id`

3. **Run Update Job**:
   - Should automatically pick up incident from previous job
   - Add `work_notes`: "Incident updated via workflow"
   - Set `new_state`: "6" (Resolved)

4. **Verify in ServiceNow**:
   - Check that incident was created and updated correctly

### Expected Variable Flow:
```
Create Job Output:
✓ created_incident_number: "INC0012345"
✓ created_incident_sys_id: "abc123def456"

Update Job Input:
✓ target_incident_number: "INC0012345" (from workflow artifacts)
✓ target_incident_sys_id: "abc123def456" (from workflow artifacts)
```

## Common Troubleshooting

### Issue: "not finding it" appears in debug
- **Cause**: Workflow artifacts not properly configured
- **Solution**: Ensure `aggregate: true` in `set_stats`

### Issue: Module authentication fails
- **Cause**: Incorrect module defaults structure
- **Solution**: Use `group/servicenow.itsm.servicenow:` format

### Issue: Variables still not passing
- **Check**: Job Template permissions and credential assignment
- **Check**: Workflow node connections are properly configured
- **Check**: Variable names match exactly between playbooks

## Success Verification

When working correctly, you should see:
1. ✅ Create job completes and shows incident number
2. ✅ Update job displays "Received incident number: INC######"
3. ✅ Both jobs have matching incident numbers in their logs
4. ✅ ServiceNow shows the incident with proper creation and update history
