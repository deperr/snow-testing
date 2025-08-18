# ServiceNow ITSM Playbooks for Ansible Automation Platform 2.5

This directory contains playbooks for managing ServiceNow incidents through Ansible Automation Platform 2.5.

## Playbooks

### 1. snow_create_incident.yml
Creates a new ServiceNow incident with configurable parameters.

**Purpose**: Initial incident creation in a workflow
**Required Variables**: 
- `incident_title` (or defaults to "Automated Incident via Ansible")

**Optional Variables**:
- `incident_details`: Detailed description
- `urgency`: 1=High, 2=Medium, 3=Low (default: 3)
- `impact`: 1=High, 2=Medium, 3=Low (default: 3)
- `priority`: 1=Critical, 2=High, 3=Moderate, 4=Low, 5=Planning (default: 5)
- `category`: Incident category (default: "Software")
- `subcategory`: Incident subcategory
- `caller`: Caller ID
- `assignment_group`: Assignment group
- `assigned_to`: Assigned user

**Outputs**: 
- Sets `created_incident_number`, `created_incident_sys_id` as facts
- Saves incident details to job artifacts for workflow use

### 2. snow_update_incident.yml
Updates an existing ServiceNow incident.

**Purpose**: Update incident status, assignment, or other fields later in workflow
**Required Variables**:
- `incident_number` OR `incident_sys_id` (to identify the incident)

**Optional Variables** (only defined fields will be updated):
- `new_short_description`: Updated short description
- `new_description`: Updated description
- `new_urgency`: Updated urgency
- `new_impact`: Updated impact
- `new_priority`: Updated priority
- `new_state`: Updated state (1=New, 2=In Progress, 6=Resolved, 7=Closed)
- `new_category`: Updated category
- `new_assignment_group`: Updated assignment group
- `new_assigned_to`: Updated assigned user
- `work_notes`: Work notes to add
- `close_notes`: Notes when closing
- `close_code`: Close code when resolving/closing

**Outputs**:
- Sets `updated_incident_*` facts
- Saves update details to job artifacts

## AAP Credential Configuration

These playbooks use module defaults and expect ServiceNow credentials to be configured in AAP:

1. Create a **ServiceNow** credential type in AAP
2. Configure the following fields:
   - `ansible_servicenow_instance`: Your ServiceNow instance (e.g., "dev12345.service-now.com")
   - `ansible_servicenow_username`: ServiceNow username
   - `ansible_servicenow_password`: ServiceNow password

3. Assign this credential to your job templates

## Workflow Template Setup

### Example Workflow Sequence:
1. **Create Incident** (`snow_create_incident.yml`)
   - Survey variables: `incident_title`, `incident_details`, `urgency`, `impact`
   - Creates incident and sets facts

2. **[Your Custom Tasks]**
   - Perform automation tasks
   - Gather results

3. **Update Incident** (`snow_update_incident.yml`)
   - Use `{{ created_incident_number }}` from step 1
   - Update with results: `new_state`, `work_notes`, etc.

## Survey Configuration Examples

### For Create Incident Job Template:
```yaml
- variable: incident_title
  question: "Incident Title"
  type: text
  required: true

- variable: incident_details  
  question: "Incident Description"
  type: textarea
  required: false

- variable: urgency
  question: "Urgency Level"
  type: multiplechoice
  choices: ["1", "2", "3"]
  default: "3"

- variable: impact
  question: "Impact Level"
  type: multiplechoice
  choices: ["1", "2", "3"] 
  default: "3"
```

### For Update Incident Job Template:
```yaml
- variable: incident_number
  question: "Incident Number (from previous step)"
  type: text
  required: true

- variable: new_state
  question: "New State"
  type: multiplechoice
  choices: ["2", "6", "7"]
  default: "2"

- variable: work_notes
  question: "Work Notes"
  type: textarea
  required: false
```

## Logging

Both playbooks log activities to `/tmp/servicenow_incidents.log` by default. Set `log_incidents: false` to disable logging.

## Best Practices

1. **Workflow Variables**: Use the artifacts from the create playbook in subsequent update calls
2. **Error Handling**: Both playbooks include validation and will fail gracefully with clear error messages
3. **Idempotency**: Update playbook only modifies fields that are explicitly provided
4. **Security**: Always use AAP credentials rather than hardcoded values
5. **Monitoring**: Check job artifacts for incident URLs and details

## Dependencies

- `servicenow.itsm` collection must be installed in your execution environment
- ServiceNow instance must be accessible from AAP execution nodes
- Proper ServiceNow user permissions for incident creation/modification
