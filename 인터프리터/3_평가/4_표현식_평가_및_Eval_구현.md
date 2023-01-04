## 표현식 평가

Eval 함수의 기본 시그니처: 모든 노드를 매개변수로 전달 받을 수 있도록 설정

```go
func Eval(node ast.Node) object.Object
```

### Eval 구현

```go
package evaluator

import (
	"monkey/ast"
	"monkey/object"
)

func Eval(node ast.Node) object.Object {
	switch node := node.(type) {

	case *ast.Program:
		return evalProgram(node, env)

	case *ast.ExpressionStatement:
		return Eval(node.Expression, env)

	case *ast.IntegerLiteral:
		return &object.Integer{Value: node.Value}
    }

	return nil
}

func evalProgram(program *ast.Program) object.Object {
	var result object.Object

    for _, statement := range program.Statements {
		result = Eval(statement)
	}
	return result
}
```

### REPL 완성하기

```go
package repl

import (
	"bufio"
	"fmt"
	"io"
	"monkey/evaluator"
	"monkey/lexer"
	"monkey/object"
	"monkey/parser"
)

const PROMPT = ">> "

func Start(in io.Reader, out io.Writer) {
	scanner := bufio.NewScanner(in)

	for {
		fmt.Printf(out, PROMPT)
		scanned := scanner.Scan()
		if !scanned {
			return
		}

		line := scanner.Text()
		l := lexer.New(line)
		p := parser.New(l)

		// program을 parse하고 Eval에 넘기기!
		program := p.ParseProgram()
		if len(p.Errors()) != 0 {
			printParserErrors(out, p.Errors())
			continue
		}

		evaluated := evaluator.Eval(program)
		// nil이 아닌 object.Object 반환시 이를 출력
		if evaluated != nil {
			io.WriteString(out, evaluated.Inspect())
			io.WriteString(out, "\n")
		}
	}
}
```
