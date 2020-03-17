### 循环遍历

```javascript
let flag = false
let checkSpecCodeArr = []
// for (var k = 0; k < list.length; k++) {
//   if (list[k].shelfNum === 0) {
//     flag = true
//     specCode = list[k].specCode
//     return false
//   }
// }
list.forEach(a => {
  if (a.shelfNum === 0) {
    flag = true
    checkSpecCodeArr.push(a.specCode)
  }
})
```

