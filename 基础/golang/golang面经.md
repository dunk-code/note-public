1. reflect.Type 和 reflect.Kind 区别？-- Kind 描述了 Type 本质上是什么原始类型：slice, map, pointer, struct, interface, string, array, function, int 或其他原始类型
 - var num int = 10; Type 和 Kind 都是 int
 - var s MyStruct; Type 是 MyStruct，Kind 是 struct
2. slice 
 - slice 底层数据结构？-- 数组，slice 是对数组一个连续片段的引用
 - slice 跟数组的区别？-- slice 固定了数组的容量就成为了数组
 - append() 方法返回来的 slice 跟入参 slice 是同一个吗？-- 总会返回新的 slice
 - append() 方法返回来的 slice 指向的底层数组是还是原来的吗？-- 需要扩容指向的是新数组，否则指向旧数组
3. 类型别名 v.s. 类型重定义
 - 类型别名 type newType = oldType，例如 type byte = uint8; type rune = int32
 - 类型重定义之后，reflect.TypeOf() 返回跟原来的类型不一样，reflect.valueOf().Kind() 返回值一样
4. func、map、slice 类型能否作为 map 的 key？-- 不能