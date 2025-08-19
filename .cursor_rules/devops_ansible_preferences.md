# DevOps & Ansible Automation Coding Preferences

## 🎯 **Core Identity & Expertise**
I am a Senior DevOps Engineer and Backend Solutions Developer with expertise in:
- Kubernetes and Azure Pipelines
- Python and Bash scripting  
- Ansible automation and configuration management
- ServiceNow ITSM integration
- Azure Cloud Services architecture
- Amazon Web Services architecture
- GitHub Actions
- GitLab Runners

## 📋 **Ansible Playbook Structure & Style**

### **Task Organization Principles**
```yaml
# ✅ ALWAYS follow this task structure order:
- name: Descriptive task name
  # 1. Task metadata FIRST (in this order):
  register: variable_name
  when: conditions
  loop: items
  vars: local_variables
  delegate_to: host
  run_once: true
  # 2. Module call LAST:
  module.name:
    parameters: values
```

### **Variable Management**
- **Eliminate redundant vars sections** when possible
- **Apply defaults inline** where variables are used
- **Use set_fact tasks** for complex variable logic instead of vars sections
- **Keep vars sections minimal** - only for variables used in multiple tasks

### **Naming Conventions**
- **camelCase**: Variables, functions, method names
- **PascalCase**: Class names
- **snake_case**: File names, directory structures, Ansible variables
- **UPPER_CASE**: Environment variables
- **Descriptive task names**: Clear, action-oriented descriptions

### **Module Defaults Best Practices**
```yaml
# ✅ Use proper module defaults structure
module_defaults:
  servicenow.itsm.incident:
    instance:
      host: "{{ servicenow_instance }}"
      username: "{{ servicenow_username }}"
      password: "{{ servicenow_password }}"
  servicenow.itsm.incident_info:
    instance:
      host: "{{ servicenow_instance }}"
      username: "{{ servicenow_username }}"
      password: "{{ servicenow_password }}"
```

## 🔄 **Workflow Integration Patterns**

### **Variable Passing Between Playbooks**
```yaml
# ✅ Use set_stats for workflow artifact sharing
- name: Save details to job artifacts
  ansible.builtin.set_stats:
    data:
      created_incident_number: "{{ result.record.number }}"
      created_incident_sys_id: "{{ result.record.sys_id }}"
    per_host: false
    aggregate: true  # CRITICAL for workflow sharing
```

### **Smart Variable Priority Logic**
```yaml
# ✅ Implement cascading variable resolution
target_incident_number: "{{ created_incident_number | default(updated_incident_number | default(incident_number | default(''))) }}"
```

## 🛡️ **Error Handling & Validation**

### **Comprehensive Validation**
```yaml
# ✅ Always validate inputs and states
- name: Validate required parameters
  ansible.builtin.assert:
    that:
      - required_var is defined
      - required_var | length > 0
    fail_msg: "Clear, actionable error message"
    success_msg: "Validation successful"
```

### **Graceful Fallbacks**
```yaml
# ✅ Implement fallback logic for robustness
- name: Set facts with fallback
  when: primary_result is not defined or primary_result.record is not defined
  ansible.builtin.set_fact:
    fallback_values: "{{ default_values }}"
```

## 📝 **Documentation Standards**

### **Code Comments**
- **Clear purpose statements** at the top of playbooks
- **Inline comments** for complex logic
- **State reference comments** for ServiceNow values
- **Priority comments** for variable resolution logic

### **README Structure**
- **Purpose and features** clearly stated
- **Usage examples** with multiple scenarios
- **Variable reference** with all options
- **AAP integration** instructions
- **Troubleshooting** section with common issues

## 🚀 **Enterprise & Production Considerations**

### **Avoid Common Pitfalls**
```yaml
# ❌ NEVER use ansible_date_time with gather_facts: false
# ✅ Use alternative timestamp methods:
current_timestamp: "{{ lookup('pipe', 'date -u +%Y-%m-%dT%H:%M:%SZ') }}"
```

### **Red Hat Certified Content Preference**
- **Prefer Red Hat certified collections** over community versions
- **Use redhat.controller_configuration** instead of awx.awx
- **Specify execution environments** for enterprise deployments
- **Document certification benefits** and enterprise value

### **Security Best Practices**
- **Use AAP credentials** instead of hardcoded values
- **Implement least privilege** access patterns
- **Encrypt sensitive data** with Ansible Vault
- **Validate certificates** by default

## 🔧 **Automation & Infrastructure Patterns**

### **Idempotent Design**
- **All operations must be idempotent** and safe to re-run
- **Use proper state management** (present/absent/specific states)
- **Implement check mode support** where applicable

### **Modular Architecture**
- **Break complex tasks** into focused, single-purpose playbooks
- **Design for workflow integration** from the start
- **Create reusable components** with clear interfaces
- **Maintain backward compatibility** in interfaces

### **Performance Optimization**
- **Use gather_facts: false** unless specifically needed
- **Parallel tool calls** when possible for information gathering
- **Efficient variable handling** to reduce overhead
- **Batch operations** where supported

## 📊 **Testing & Quality Assurance**

### **Validation Requirements**
- **Syntax validation** with ansible-playbook --syntax-check
- **Linting compliance** with no errors
- **End-to-end testing** in development environments
- **Variable flow testing** in workflow scenarios

### **Code Organization**
- **Logical file structure** with clear separation of concerns
- **Consistent formatting** and indentation
- **Version control friendly** structure and naming

## 🎯 **Specific Technology Preferences**

### **ServiceNow Integration**
- **Use servicenow.itsm collection** for all ITSM operations
- **Implement proper state management** (1=New, 2=In Progress, 6=Resolved, 7=Closed)
- **Include work notes and activity logging** for audit trails
- **Handle both workflow and standalone** operation modes

### **AAP Integration**
- **Design for survey compatibility** with clear variable structures
- **Implement proper artifact sharing** between workflow steps
- **Use execution environments** for consistent runtime
- **Include comprehensive error handling** for production use

### **Azure & Cloud**
- **Follow Azure best practices** for resource management
- **Use managed identities** for authentication where possible
- **Implement proper tagging** and resource organization
- **Design for high availability** and disaster recovery

## 💡 **Decision-Making Philosophy**

### **When to Optimize**
- **Eliminate redundancy** in code structure
- **Simplify complex logic** while maintaining functionality
- **Prefer explicit over implicit** behavior
- **Optimize for maintainability** over brevity

### **When to Add Complexity**
- **Enterprise security requirements** always take priority
- **Comprehensive error handling** is worth the extra code
- **Workflow integration** justifies additional complexity
- **Production reliability** over development convenience

### **Code Review Priorities**
1. **Functionality correctness** and error handling
2. **Security and compliance** considerations
3. **Maintainability and readability**
4. **Performance and efficiency**
5. **Documentation completeness**

## 🔄 **Continuous Improvement**

### **Refactoring Principles**
- **Regularly eliminate redundant patterns**
- **Consolidate common logic** into reusable components
- **Update to latest certified collections** and best practices
- **Simplify complex variable handling** when possible

### **Learning & Adaptation**
- **Stay current** with Red Hat and Ansible best practices
- **Incorporate feedback** from production deployments
- **Document lessons learned** for future reference
- **Share knowledge** through clear documentation and examples

---

## 🎯 **Quick Reference Checklist**

When writing Ansible code, ensure:
- ✅ Task metadata before module calls
- ✅ Minimal or no vars sections
- ✅ Comprehensive error handling
- ✅ Workflow artifact compatibility
- ✅ Red Hat certified collections
- ✅ No ansible_date_time with gather_facts: false
- ✅ Clear, descriptive task names
- ✅ Proper module defaults structure
- ✅ Documentation for complex logic
- ✅ Enterprise security considerations
