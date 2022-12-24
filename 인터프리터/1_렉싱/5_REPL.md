## 첫 번째 REPL

`REPL(Read-Eval-Print Loop)`: 단일 사용자의 입력을 받아서 이를 평가하고, 그 결과를 사용자에게 다시 반환하는 단순한 컴퓨터 프로그래밍 환경.

입력을 읽고(Read), 인터프리터에게 보내 평가하고(Eval), 인터프리터의 결과물을 출력(Print)하는 일련의 과정을 반복(Loop)

- Python, JavaScript 런타임, Ruby, Lisp류 언어들 모두 REPL을 지님
- `콘솔`, `대화형(interactive) 모드`

### 기본 구현

생성된 토큰을 그대로 출력하는 기초 REPL

```go
package main

import (
	"fmt"
	"monkey/repl"
	"os"
	"os/user"
)

func main() {
	user, err := user.Current()
	if err != nil {
		panic(err)
	}
	fmt.Printf("Hello %s! This is the Monkey programming language!\n",
		user.Username)
	fmt.Printf("Feel free to type in commands\n")
	repl.Start(os.Stdin, os.Stdout)
}

```

```go
package repl

import (
	"bufio"
	"fmt"
	"io"
	"monkey/lexer"
	"monkey/token"
)

const PROMPT = ">> "

func Start(in io.Reader, out io.Writer) {
	scanner := bufio.NewScanner(in)

	for {
		fmt.Printf(PROMPT)
		scanned := scanner.Scan()
		if !scanned {
			return
		}

		line := scanner.Text()
		// 입력값에 대해 렉서 생성
		l := lexer.New(line)

		// 렉서로 한줄한줄 읽으며 토큰 생성하여 출력
		for tok := l.NextToken(); tok.Type != token.EOF; tok = l.NextToken() {
			fmt.Printf("%+v\n", tok)
		}
	}
}
```

- 예시 출력

```sh
Hello jeong! This is the Monkey programming language!
Feel free to type in commands
>> let x = 10;
{Type:LET Literal:let}
{Type:IDENT Literal:x}
{Type:= Literal:=}
{Type:INT Literal:10}
{Type:; Literal:;}
```
