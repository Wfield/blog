`var answer = 6 * 7;`

Syntax:
```
{
  "type": "Program",  // 描述该语句的类型 
  "body": [
    {
      "type": "VariableDeclaration",
      "declarations": [   // 声明的内容数组，里面的每一项也是一个对象
        {
          "type": "VariableDeclarator",   // 描述该语句的类型 
          "id": {      //  描述变量名称的对象
            "type": "Identifier",       // 类型
            "name": "answer"   // 变量的名字
          },
          "init": {    // 初始化变量值对象
            "type": "BinaryExpression",
            "operator": "*",
            "left": {
              "type": "Literal",
              "value": 6,
              "raw": "6"
            },
            "right": {
              "type": "Literal",
              "value": 7,
              "raw": "7"
            }
          }
        }
      ],
      "kind": "var"   // 变量声明的关键字
    }
  ],
  "sourceType": "script"
}
```
Token:
```
[
    {
        "type": "Keyword",
        "value": "var"
    },
    {
        "type": "Identifier",
        "value": "answer"
    },
    {
        "type": "Punctuator",
        "value": "="
    },
    {
        "type": "Numeric",
        "value": "6"
    },
    {
        "type": "Punctuator",
        "value": "*"
    },
    {
        "type": "Numeric",
        "value": "7"
    },
    {
        "type": "Punctuator",
        "value": ";"
    }
]
```
ast 转换网站：[https://esprima.org/demo/parse.html#](https://esprima.org/demo/parse.html#)
