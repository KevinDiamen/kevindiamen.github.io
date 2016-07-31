---
layout: post
title: AIDL 学习笔记
category: android
tags: [android]
---

    array array_splice ( array &$input , int $offset int $length = 0 bool $preserve_keys ] ) 有四个参数 第一个是输入数组，第二个是偏移量 ，第三个是截取长度默认是input的长度, 第四个是bool代表返回的数组是否保留之前的key


    PHP_FUNCTION(array_slice)
    {
    zval     *input,        /* 输入的数组 */
        **z_length = NULL, /* 数组长度 */ 
        **entry;        
    long     offset,        /* 起始偏移位置 */
    length = 0;    /* 返回数组长度 */
    zend_bool preserve_keys = 0; /* 是否重置索引 默认不重置 */
    int      num_in,        /* 输入数组的长度 */
             pos;           /* 当前的指针 */
    char *string_key;
    uint string_key_len;
    ulong num_key;
    HashPosition hpos;

    /* 判断参数个数和类型是否正确 */
    if (zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "al|Zb", &input, &offset, &z_length, &preserve_keys) == FAILURE) {
    return;
    }

    /*  输入的数组的元素个数 */
    num_in = zend_hash_num_elements(Z_ARRVAL_P(input));

    /* 如果没有传第三个参数 让length = num_in */
    if (ZEND_NUM_ARGS() < 3 || Z_TYPE_PP(z_length) == IS_NULL)     {
    length = num_in;
    } else {
    convert_to_long_ex(z_length); /* 强制转换z_length 为 long 类型 */
    length = Z_LVAL_PP(z_length); 
    }

    /* 如果偏移量大于输入数组的长度 返回null */
    if (offset > num_in) {
      array_init(return_value);
      return;
    } else if (offset < 0 && (offset = (num_in + offset)) < 0) {
      offset = 0; /* offset 可以为负数 但是如果 offset 加上 输入数组的长度小于0 就让offset = 0  同时也让offset 为>=0*/
    }

    /* 计算length 的长度 */
    if (length < 0) {
      length = num_in - offset + length;
    } else if (((unsigned long) offset + (unsigned long) length) > (unsigned) num_in) {
      length = num_in - offset;
    }

    /* 初始化返回数组 */
    array_init_size(return_value, length > 0 ? length : 0);

    if (length <= 0) {
        return;
    }

    /* 把pos 指向 数组input中 offset处 */
    pos = 0;
    zend_hash_internal_pointer_reset_ex(Z_ARRVAL_P(input), &hpos);
    while (pos < offset && zend_hash_get_current_data_ex(Z_ARRVAL_P(input), (void **)&entry, &hpos) == SUCCESS) {
        pos++;
        zend_hash_move_forward_ex(Z_ARRVAL_P(input), &hpos);
    }

    /* 把元素复制到return_value上 */
    while (pos < offset + length && zend_hash_get_current_data_ex(Z_ARRVAL_P(input), (void **)&entry, &hpos) == SUCCESS) {

    zval_add_ref(entry);

    switch (zend_hash_get_current_key_ex(Z_ARRVAL_P(input), &string_key, &string_key_len, &num_key, 0, &hpos)) {
        case HASH_KEY_IS_STRING:
            zend_hash_update(Z_ARRVAL_P(return_value), string_key, string_key_len, entry, sizeof(zval *), NULL);
            break;

        case HASH_KEY_IS_LONG:
            if (preserve_keys) {
                zend_hash_index_update(Z_ARRVAL_P(return_value), num_key, entry, sizeof(zval *), NULL);
            } else {
                zend_hash_next_index_insert(Z_ARRVAL_P(return_value), entry, sizeof(zval *), NULL);
            }
            break;
    }
    pos++;
    zend_hash_move_forward_ex(Z_ARRVAL_P(input), &hpos);
    }
    }
#####下面是用PHP翻译过来的

    function slice($input,$offset,$length,$preserve_keys)
    {
    $num_in = count($input);

    if ($length === null) {
        $length = $num_in;
    }

    if ($offset < 0 && ($offset = ($num_in + $offset)) < 0) {
        $offset = 0;
    } elseif ($offset > $num_in) {
        return;
    }

    if ($length > 0 && ($length + $offset) > $num_in) {
        $length = $num_in - $offset;
    } elseif ($length < 0) {
        $length = $num_in - $offset + $length;
    }

    while ($offset--) {
        next($input);
    }

    $result_array = array();
    while ($length--) {
        if ($preserve_keys) {
            $result_array[key($input)] = current($input);
        } else {
            $result_array[] = current($input);
        }
        next($input);
    }

     return $result_array;
    }