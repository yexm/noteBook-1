GO LANG

基础语法：
    变量初始化：
        使用表达式赋值变量，不需要指明变量类型

        在函数中，`:=` 简洁赋值语句在明确类型的地方，可以用于替代 var 定义，函数外的每个语句都必须以关键字开始（`var`、`func`、等等），`:=` 结构不能使用在函数外。

    Go的基本数据类型：
        bool
        string
        int int8 int16 int32 int64
        uint uint8 uint16 uint32 uint64 uintptr
        float32 float64
        complex64 complex128

        零值，数值类型“0”， 布尔值类型“False”， 字符串类型“”

        类型转换表达式：
            表达式 T(v) 将值 v 转换为类型 `T`。与C不同，不同类型赋值需要显示转换类型。

        常量：
            const关键字定义

逻辑：
    for
        与C和JAVA一样，但是for强制不可使用（）来圈定循环条件
        循环条件中前置与后置语句可以为空
        没有while，因为形势一样

    if 
        判断条件强制不使用（）
        可在判断条件前增加类似for的前置语句，语句中定义的变量作用域尽在if语句内

    switch
        