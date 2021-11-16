# set_fact

从任务中设置主机 facts。

- 该模块允许设置新的变量。
- 变量是在逐个主机的基础上设置的，就像 `setup` 模块所发现的 facts 一样。
- 在ansible playbook运行期间，这些变量将可用于后续的 plays。
- 将`cacheable`设置为yes，以使用 facts 缓存跨执行保存变量。使用`set_fact`创建的变量具有不同的优先级，这取决于它们是否被缓存。
- 根据标准Ansible变量优先级规则，许多其他类型的变量具有更高的优先级，因此此值可能会被覆盖。
- Windows目标也支持此模块。
  
## 参数

- cacheable     boolean

    可选值：no / yes  
    此布尔值将变量转换为实际的 'fact'，如果启用了事实缓存，该 'fact' 也将添加到fact缓存中。  
    通常，此模块会创建**主机级变量**，并且具有更高的优先级，此选项会更改所创建变量的性质和优先级（通过7个步骤）。  
    这实际上创建了两个变量的副本，一个是普通的 "set_fact" 主机变量，具有较高的优先级，另一个是较低的 "ansible_fact"，可通过facts缓存插件进行持久化。这与 `meta: clear_facts` 可能产生混淆，因为它将删除'ansible_fact'而不是主机变量。

- key_value     -/required
  
    set_fact 模块将key=value对**作为变量**设置在playbook范围内。或者，使用args:语句接受复杂参数。

## 示例

```yaml
# Example setting host facts using key=value pairs, note that this always creates strings or booleans
- set_fact: one_fact="something" other_fact="{{ local_var }}"

# Example setting host facts using complex arguments
- set_fact:
     one_fact: something
     other_fact: "{{ local_var * 2 }}"
     another_fact: "{{ some_registered_var.results | map(attribute='ansible_facts.some_fact') | list }}"

# Example setting facts so that they will be persisted in the fact cache
- set_fact:
    one_fact: something
    other_fact: "{{ local_var * 2 }}"
    cacheable: yes

# As of Ansible 1.8, Ansible will convert boolean strings ('true', 'false', 'yes', 'no')
# to proper boolean values when using the key=value syntax, however it is still
# recommended that booleans be set using the complex argument style:
- set_fact:
    one_fact: yes
    other_fact: no
```
