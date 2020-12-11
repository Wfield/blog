## 原理
Lazyjs 对不同的计算有不同的计算模式：
  例如，MappedArrayWrapper、MappedSequence、IndexedMappedSequence 都是对应 map 计算模式
而求值的过程，或者说求值的模式是统一的，都是借助 Sequence 链，链条上的每个 Sequence 节点只负责：
- 上传：向上级 Sequence 获取值
- 下达：向下级 Sequence 传递至
- 求值：应用类型对应的计算模式对数据进行计算或者过滤等操作

**取值的过程是从 Sequence 链的下方至上方获取值，然后沿着链返回值**

```
var seq0 = Lazy([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
var seq1 = seq0.map(i => i * 2)
var seq2 = seq1.filter(i => i <= 10)
var seq3 = seq2.take(3)

// 求值
seq3.each(i => print(i))
```
获得第一个最终结果值的过程为：
```
[1] seq3 调用 seq2 获取第 1 个值
[2] seq2 调用 seq1 获取第 1 个值
[3] seq1 直接从 seq0 的 source 属性对应的原始数值取值（第 1 个值为  1）
[4] seq1 得到 1，应用计算（i => i * 2），得到 2，返回给调用方（seq2）
[5] seq2 得到 2，应用计算（i => i < 10），满足条件，将 2 返回给调用方（seq3）
[6] seq3 得到 2，应用计算（已返回值数目是否小于 3），满足条件，将 2 返回给调用方（seq3.each）
```

## MappedSequence 的实现 （对应 map 方法）
```
function MappedSequence(parent, mapFn) {
  var seq = new Sequence()
  seq.getIterator = () => {
    var iterator = parent.getIterator()
    var index = -1
    return {
      current() { return mapFn(iterator.current(), index) },
      moveNext() {
        if (iterator.moveNext()) {
          ++index
          return true
        }
        return false
      }
    }
  }
  seq.each = fn => parent.each((e, i) => fn(mapFn(e, i), i))
  return seq
}
```
在定义中只定义了从 parent 获取值的**方法**，和读取自身值的**方法**；没有执行对值的操作；


参考链接：https://zhuanlan.zhihu.com/p/24138694
