## Return 문

- return문은 일련의 명령문에 대한 평가를 멈추면서 해당 블록을 탈출.

1. 특정 값이 return문에 의해 반환되었는지의 여부를 판별해야 하므로 return object 구현 필요.

```go
package object

const RETURN_VALUE_OBJ = "RETURN_VALUE"

// 반환값을 감싸는 wrapper object
type ReturnValue struct {
	Value Object
}

func (rv *ReturnValue) Type() ObjectType { return RETURN_VALUE_OBJ }
func (rv *ReturnValue) Inspect() string  { return rv.Value.Inspect() }
```

2. 블록이 중첩된 상황에서 안쪽 블록의 return문이 실행되는 경우, 바깥쪽 블록의 평가도 중단되어야하므로 그대로 `evalBlockStatement`에서는 `ReturnValue`를 그대로 반환. `evalProgram`까지 도달했을 때 그 반환값을 unwrap.

```go
// evaluator.go
func Eval(node ast.Node, env *object.Environment) object.Object {
	switch node := node.(type) {
		// ...
		case *ast.ReturnStatement:
		val := Eval(node.ReturnValue, env)
		if isError(val) {
			return val
		}
		return &object.ReturnValue{Value: val}
	}
	return nil
}

func evalProgram(program *ast.Program, env *object.Environment) object.Object {
	var result object.Object

	for _, statement := range program.Statements {
		result = Eval(statement, env)

		switch result := result.(type) {
		case *object.ReturnValue:
			return result.Value // unwrap
		case *object.Error:
			return result
		}
	}

	return result
}

func evalBlockStatement(
	block *ast.BlockStatement,
	env *object.Environment,
) object.Object {
	var result object.Object

	for _, statement := range block.Statements {
		result = Eval(statement, env)

		if result != nil {
			rt := result.Type()
			if rt == object.RETURN_VALUE_OBJ || rt == object.ERROR_OBJ {
				return result
			}
		}
	}

	return result
}
```
