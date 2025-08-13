# ServiceNow Ansible Variable Passing Guide

This guide demonstrates various patterns for passing values between tasks in ServiceNow Ansible playbooks, particularly for creating tickets and using the returned values for updates.

## Overview

The playbooks in this directory demonstrate four main patterns for variable passing:

1. **Simple Create-and-Update Workflow** (`snow_create_and_update_incident.yml`)
2. **Advanced Variable Passing Patterns** (`snow_variable_passing_examples.yml`)
3. **Master Workflow with Sub-playbooks** (`snow_master_workflow.yml`)
4. **Sub-playbook Examples** (`sub_create_incidents.yml`, `sub_update_incident.yml`)

## Pattern 1: Basic Create-and-Update Workflow

### Usage
```bash
# Basic workflow - create and update in sequence
ansible-playbook playbooks_itsm/snow_create_and_update_incident.yml \
  -e servicenow_instance="your-instance.service-now.com" \
  -e servicenow_username="your-username" \
  -e servicenow_password="your-password" \
  -e incident_short_description="Test incident for workflow"

# Disable auto-update
ansible-playbook playbooks_itsm/snow_create_and_update_incident.yml \
  -e enable_auto_update=false \
  -e servicenow_instance="your-instance.service-now.com" \
  -e servicenow_username="your-username" \
  -e servicenow_password="your-password"

# Create related change request
ansible-playbook playbooks_itsm/snow_create_and_update_incident.yml \
  -e create_related_change_request=true \
  -e servicenow_instance="your-instance.service-now.com" \
  -e servicenow_username="your-username" \
  -e servicenow_password="your-password"
```

### Key Variables Created
```yaml
# Available after incident creation
incident_sys_id: "abc123..."
incident_number: "INC0000123"
incident_url: "https://instance.service-now.com/nav_to.do?uri=incident.do?sys_id=abc123..."

# Available after update
final_incident_state: "in_progress"
final_assignment_group: "IT Support"
workflow_completed_at: "2024-01-01T12:00:00Z"
```

## Pattern 2: Advanced Variable Passing

### Batch Operations
```bash
# Create multiple incidents and track results
ansible-playbook playbooks_itsm/snow_variable_passing_examples.yml \
  -e create_batch_incidents=true \
  -e servicenow_instance="your-instance.service-now.com" \
  -e servicenow_username="your-username" \
  -e servicenow_password="your-password"
```

### Conditional Updates
```bash
# Apply conditional updates based on incident age
ansible-playbook playbooks_itsm/snow_variable_passing_examples.yml \
  -e apply_conditional_updates=true \
  -e target_incident_sys_id="existing-incident-sys-id" \
  -e servicenow_instance="your-instance.service-now.com" \
  -e servicenow_username="your-username" \
  -e servicenow_password="your-password"
```

### External Variable Integration
```bash
# Use external variables for dynamic updates
ansible-playbook playbooks_itsm/snow_variable_passing_examples.yml \
  -e use_external_variables=true \
  -e external_incident_sys_id="existing-incident-sys-id" \
  -e external_incident_updates='{"urgency": "high", "assignment_group": "L2 Support"}' \
  -e servicenow_instance="your-instance.service-now.com" \
  -e servicenow_username="your-username" \
  -e servicenow_password="your-password"
```

### Lifecycle Tracking
```bash
# Track complete incident lifecycle
ansible-playbook playbooks_itsm/snow_variable_passing_examples.yml \
  -e track_incident_lifecycle=true \
  -e lifecycle_incident_description="Lifecycle tracking test" \
  -e servicenow_instance="your-instance.service-now.com" \
  -e servicenow_username="your-username" \
  -e servicenow_password="your-password"
```

## Pattern 3: Master Workflow with Sub-playbooks

### Basic Master Workflow
```bash
# Run complete master workflow
ansible-playbook playbooks_itsm/snow_master_workflow.yml \
  -e servicenow_instance="your-instance.service-now.com" \
  -e servicenow_username="your-username" \
  -e servicenow_password="your-password"
```

### Advanced Master Workflow Options
```bash
# Create multiple incidents and export results
ansible-playbook playbooks_itsm/snow_master_workflow.yml \
  -e create_multiple_incidents=true \
  -e incident_creation_count=3 \
  -e export_results=true \
  -e incident_assignment_group="L2 Support" \
  -e servicenow_instance="your-instance.service-now.com" \
  -e servicenow_username="your-username" \
  -e servicenow_password="your-password"

# Run workflow without cleanup
ansible-playbook playbooks_itsm/snow_master_workflow.yml \
  -e cleanup_shared_vars=false \
  -e servicenow_instance="your-instance.service-now.com" \
  -e servicenow_username="your-username" \
  -e servicenow_password="your-password"
```

## Key Variable Passing Techniques

### 1. Using `register` to Capture Task Results
```yaml
- name: Create incident
  servicenow.itsm.incident:
    short_description: "Test incident"
  register: created_incident

- name: Use the created incident sys_id
  servicenow.itsm.incident:
    sys_id: "{{ created_incident.record.sys_id }}"
    state: present
    assignment_group: "IT Support"
```

### 2. Using `set_fact` to Create Variables
```yaml
- name: Extract incident details
  ansible.builtin.set_fact:
    incident_sys_id: "{{ created_incident.record.sys_id }}"
    incident_number: "{{ created_incident.record.number }}"
    incident_url: "https://{{ servicenow_instance }}/nav_to.do?uri=incident.do?sys_id={{ created_incident.record.sys_id }}"
```

### 3. Using Shared Variables Files for Inter-Playbook Communication
```yaml
# In master playbook
- name: Create shared variables file
  ansible.builtin.copy:
    content: |
      workflow_id: "{{ workflow_id }}"
      created_incidents: []
    dest: "/tmp/shared_vars.yml"

# In sub-playbook
- name: Load shared variables
  ansible.builtin.include_vars:
    file: "/tmp/shared_vars.yml"
    name: shared_data

- name: Update shared variables
  ansible.builtin.copy:
    content: "{{ updated_data | to_nice_yaml }}"
    dest: "/tmp/shared_vars.yml"
```

### 4. Using Complex Data Structures
```yaml
- name: Create incident lifecycle tracking
  ansible.builtin.set_fact:
    incident_lifecycle:
      incident_sys_id: "{{ created_incident.record.sys_id }}"
      states:
        - state: "new"
          timestamp: "{{ ansible_date_time.iso8601 }}"
        - state: "in_progress"
          timestamp: "{{ ansible_date_time.iso8601 }}"
      transitions:
        - from: "new"
          to: "in_progress"
          timestamp: "{{ ansible_date_time.iso8601 }}"
```

## Best Practices

### 1. Variable Naming Conventions
- Use descriptive names: `incident_sys_id` instead of `id`
- Use prefixes for grouped variables: `incident_`, `workflow_`, `external_`
- Use snake_case for variable names

### 2. Error Handling
```yaml
- name: Verify incident creation
  ansible.builtin.assert:
    that:
      - created_incident is succeeded
      - created_incident.record.sys_id is defined
    fail_msg: "Incident creation failed or sys_id missing"
```

### 3. Conditional Logic
```yaml
- name: Update incident only if creation succeeded
  servicenow.itsm.incident:
    sys_id: "{{ incident_sys_id }}"
    assignment_group: "IT Support"
  when:
    - incident_sys_id is defined
    - auto_update_enabled | default(true) | bool
```

### 4. Documentation in Variables
```yaml
vars:
  # Workflow control - set to false to skip auto-updates
  auto_update_enabled: true
  
  # Incident configuration - override with -e parameters
  incident_config:
    urgency: "medium"    # low, medium, high
    impact: "medium"     # low, medium, high
    category: "Software" # Software, Hardware, Network
```

## Common Use Cases

### 1. Incident-to-Change Request Workflow
```bash
# Create incident, then create related change request
ansible-playbook playbooks_itsm/snow_create_and_update_incident.yml \
  -e create_related_change_request=true \
  -e incident_short_description="Database performance issue" \
  -e servicenow_instance="your-instance.service-now.com" \
  -e servicenow_username="your-username" \
  -e servicenow_password="your-password"
```

### 2. Bulk Incident Processing
```bash
# Create multiple incidents and process them
ansible-playbook playbooks_itsm/snow_variable_passing_examples.yml \
  -e create_batch_incidents=true \
  -e servicenow_instance="your-instance.service-now.com" \
  -e servicenow_username="your-username" \
  -e servicenow_password="your-password"
```

### 3. Automated Escalation Workflow
```bash
# Apply conditional escalation based on incident age
ansible-playbook playbooks_itsm/snow_variable_passing_examples.yml \
  -e apply_conditional_updates=true \
  -e target_incident_sys_id="existing-sys-id" \
  -e incident_age_hours=5 \
  -e servicenow_instance="your-instance.service-now.com" \
  -e servicenow_username="your-username" \
  -e servicenow_password="your-password"
```

## Troubleshooting

### Common Issues

1. **Variable Not Found**
   - Check variable spelling and case sensitivity
   - Ensure the task that creates the variable completed successfully
   - Use `ansible.builtin.debug` to display variable values

2. **Empty or Undefined Variables**
   - Use `default()` filters: `{{ variable_name | default('fallback_value') }}`
   - Check task conditions with `when` clauses

3. **Shared Variables Not Updating**
   - Verify file permissions on shared variables file
   - Check that `include_vars` is loading the correct file
   - Ensure variables are being saved in correct YAML format

### Debug Commands
```bash
# Run with increased verbosity
ansible-playbook playbooks_itsm/snow_create_and_update_incident.yml -v \
  -e servicenow_instance="your-instance.service-now.com" \
  -e servicenow_username="your-username" \
  -e servicenow_password="your-password"

# Run specific tags only
ansible-playbook playbooks_itsm/snow_create_and_update_incident.yml \
  --tags "create,display" \
  -e servicenow_instance="your-instance.service-now.com" \
  -e servicenow_username="your-username" \
  -e servicenow_password="your-password"

# Skip specific tags
ansible-playbook playbooks_itsm/snow_create_and_update_incident.yml \
  --skip-tags "update" \
  -e servicenow_instance="your-instance.service-now.com" \
  -e servicenow_username="your-username" \
  -e servicenow_password="your-password"
```

## Integration Examples

### With CI/CD Pipelines
```yaml
# In Azure DevOps Pipeline or GitHub Actions
- name: Create and Update ServiceNow Incident
  run: |
    ansible-playbook playbooks_itsm/snow_create_and_update_incident.yml \
      -e servicenow_instance="${{ secrets.SNOW_INSTANCE }}" \
      -e servicenow_username="${{ secrets.SNOW_USERNAME }}" \
      -e servicenow_password="${{ secrets.SNOW_PASSWORD }}" \
      -e incident_short_description="Pipeline deployment issue" \
      -e incident_assignment_group="DevOps Team"
```

### With External APIs
```yaml
# Load incident data from external API
- name: Get incident data from external system
  uri:
    url: "https://api.example.com/incidents/{{ incident_id }}"
    headers:
      Authorization: "Bearer {{ api_token }}"
  register: external_incident_data

- name: Create ServiceNow incident from external data
  servicenow.itsm.incident:
    short_description: "{{ external_incident_data.json.title }}"
    description: "{{ external_incident_data.json.description }}"
    urgency: "{{ external_incident_data.json.severity | map_severity }}"
```

This guide provides comprehensive examples for passing variables between ServiceNow Ansible tasks and playbooks. Use these patterns as starting points for your own automation workflows.
