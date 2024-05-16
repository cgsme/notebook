# assert - Asserts given expressions are true

断言给定表达式为真。

- 此模块通过可选的自定义消息断言给定的表达式为true。
- Windows目标也支持此模块。

## 参数

- fail_msg      string  

    用于失败断言的自定义消息。
    此参数在Ansible 2.7之前被称为 `msg`，现在它被重命名为 `fail_msg`，别名为 `msg`。

- quiet         boolean

    可选值：yes / no ，将此设置为 `yes` 以避免冗长的输出。  

- success_msg   string

    用于成功断言的自定义消息。

- taht          list / **required**

    和传递给 `when` 语句的相同形式的字符串表达式列表。

## 示例

```yaml
- assert: { that: "ansible_os_family != 'RedHat'" }

- assert:
    that:       # 定义了两个校验表达式
    - "'foo' in some_command_result.stdout"
    - number_of_the_counting == 3

- name: After version 2.7 both 'msg' and 'fail_msg' can customize failing assertion message
assert:
    that:
    - my_param <= 100
    - my_param >= 0
    fail_msg: "'my_param' must be between 0 and 100"
    success_msg: "'my_param' is between 0 and 100"

- name: Please use 'msg' when ansible version is smaller than 2.7
assert:
    that:
    - my_param <= 100
    - my_param >= 0
    msg: "'my_param' must be between 0 and 100"

- name: use quiet to avoid verbose output
assert:
    that:
    - my_param <= 100
    - my_param >= 0
    quiet: true
```

## 参考

- [assert module](https://docs.ansible.com/ansible/2.9/modules/assert_module.html)
