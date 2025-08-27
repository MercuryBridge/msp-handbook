# Ansible Coding Standards

## YAML format
- Do `yamllint .`
- Each line must be less than 80 characters long.
  - We can insert line breaks in `{{ ... }}`.
  - Wrap long lines using `>-`, `\`.
  - <details><summary>Example</summary>

    ```yaml
    # vars file
    vault_hostname: >-
      {{
        lookup('env', 'ANSIBLE_HASHI_VAULT_ADDR') |
        default('https://vault.example.com:8200') | 
        urlsplit('hostname')
      }}
    # -> "vault-alphanetworks.global"

    very_long_line_with_space: >-
      abcdefghijklmnopqrstuvwxyz
      abcdefghijklmnopqrstuvwxyz
      abcdefghijklmnopqrstuvwxyz
      {{ vault_hostname }}
    # -> "abcdefghijklmnopqrstuvwxyz abcdefghijklmnopqrstuvwxyz abcdefghijklmnopqrstuvwxyz vault-alphanetworks.global"

    very_long_word_without_space: "\
      abcdefghijklmnopqrstuvwxyz\
      abcdefghijklmnopqrstuvwxyz\
      abcdefghijklmnopqrstuvwxyz\
      {{ vault_hostname }}"
    # -> "abcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyzabcdefghijklmnopqrstuvwxyzvault-alphanetworks.global"

    multiline_text: |-
      abcdefghijklmnopqrstuvwxyz
      abcdefghijklmnopqrstuvwxyz
      abcdefghijklmnopqrstuvwxyz
      vault_hostname
    # -> "abcdefghijklmnopqrstuvwxyz\nabcdefghijklmnopqrstuvwxyz\nabcdefghijklmnopqrstuvwxyz\nvault_hostname"
    ```
    </details>
- Use `true` or `false` for boolean.
  - Do not use `yes`, `no`, `True` or `False`
- Indentation must be two (single-byte) spaces.
- Insert spaces appropriately.
  - After `:`
  - Before and after `|` for filter
  - After `{{` and before `}}`
- Insert blank line between tasks.

## Loops
- Use `loop` instead of `with_<lookup>` basically.
- [Use `label` directive with `loop_control`](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html#limiting-loop-output-with-label) when looping over complex data structures.

## Variables
### Where to set variables
- Use `group_vars`, `host_vars` or direct definition in hosts file.
  - These variables are automatically loaded for each host.
  - Do not use `vars_files` [keyword](https://docs.ansible.com/ansible/latest/reference_appendices/playbooks_keywords.html#play) or `include_vars` module basically.
- Refer [link](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable) for detailed behavior.

### Variable Name
- Use `snake_case_style`.
- Use prefix `<role_name>_` to role variables.
  - Use prefix `<collection_name>_` to common variables in the collection  
    and set `<collection_name>_xxx` as default for `<role_name>_xxx`.
- Use prefix `_` for variables defined by `register` or `set_fact` module.

  ```
  - ansible.builtin.ping:
    register: _ping_result

  - set_fact:
      _sample: "Sample"
  ```

### Role Variables
- Use `defaults/main.yml` for default parameter.
  ```yaml
  # sample_role/default/main.yml
  ---
  sample_role_arg_a: 10
  sample_role_arg_b: "{{ sample_collection_param_b }}"
  ```
- (Should) Validate arguments.
  - If `meta/argument_specs.yml` is present, parameters will be validated at the beginning of role execution.
    https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html#role-argument-validation
  ```yaml
  # sample_role/meta/main.yml
  ---
  argument_specs:
    # default entry point
    main:
      short_description: Sample Role
      options:
        sample_role_arg_a:
          type: "int"
          default: 10
          description: "Sample argument A"

        sample_role_arg_b:
          type: "list"
          elements: "str"
          default: "{{ sample_collection_param_b }}"
          description: "Sample argument B"

        sample_role_arg_c:
          type: "str"
          required: true
          description: "Sample argument C"
  ```


