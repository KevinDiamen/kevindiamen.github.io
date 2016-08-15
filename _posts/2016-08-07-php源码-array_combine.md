---
layout: post
title: PHP源码分析array_combine()
category: php
tags: [php]
---
PHP 源码阅读array_combine()

### PHP 源码阅读之array_combine()
- array_combine() 在 ext/standard/array.c 中

- array_combine(array $keys , array $values) — 创建一个数组，用一个数组的值作为其键名，另一个数组的值作为其值

```php
PHP_FUNCTION(array_combine)
{
zval *values, *keys;
HashPosition pos_values, pos_keys;
zval **entry_keys, **entry_values;
int num_keys, num_values;

if (zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "aa", &keys, &values) == FAILURE) { // 判断参数个数是否为2个 不是则返回FAILURE
    return;
}

num_keys = zend_hash_num_elements(Z_ARRVAL_P(keys));// 得到keys的元素数量
num_values = zend_hash_num_elements(Z_ARRVAL_P(values));

if (num_keys != num_values) {
    php_error_docref(NULL TSRMLS_CC, E_WARNING, "Both parameters should have an equal number of elements");
    RETURN_FALSE;
}

array_init_size(return_value, num_keys);//用来初始化return_value数据

if (!num_keys) {
    return;
}

zend_hash_internal_pointer_reset_ex(Z_ARRVAL_P(keys), &pos_keys);// 相当于reset()
zend_hash_internal_pointer_reset_ex(Z_ARRVAL_P(values), &pos_values);
while (zend_hash_get_current_data_ex(Z_ARRVAL_P(keys), (void **)&entry_keys, &pos_keys) == SUCCESS &&
    zend_hash_get_current_data_ex(Z_ARRVAL_P(values), (void **)&entry_values, &pos_values) == SUCCESS
) {
    if (Z_TYPE_PP(entry_keys) == IS_LONG) {
        zval_add_ref(entry_values);
        add_index_zval(return_value, Z_LVAL_PP(entry_keys), *entry_values);// 相当于result[entry_keys] = entry_values;
    } else {   //如果entry_keys 不是 IS_LONG 就都转成String 最后 result[entry_keys] = entry_values;
        zval key, *key_ptr = *entry_keys;

        if (Z_TYPE_PP(entry_keys) != IS_STRING) {
            key = **entry_keys;
            zval_copy_ctor(&key);// 复制个新内存
            convert_to_string(&key);//转成String
            key_ptr = &key;
        }

        zval_add_ref(entry_values);
        add_assoc_zval_ex(return_value, Z_STRVAL_P(key_ptr), Z_STRLEN_P(key_ptr) + 1, *entry_values);//会调用zend_hash_update（）；更新hashtable

        if (key_ptr != *entry_keys) {
            zval_dtor(&key);
        }
    }

    zend_hash_move_forward_ex(Z_ARRVAL_P(keys), &pos_keys);// 访问下一个hashtable指针
    zend_hash_move_forward_ex(Z_ARRVAL_P(values), &pos_values);
    }
}
```
