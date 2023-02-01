## 3 flag 패키지 사용

2_flag_application

### 일반적인 커맨드라인 애플리케이션 UI

형식: `application [-h] [-n <value>] -silent <arg1> <arg2>`

- 옵션: 필수적인(required) 옵션과 부가적인(optional) 옵션으로 분류됨

  - `-h`: 안내 문구 출력 여부를 지정하는 불리언 옵션값
  - `-n <value>`: n 옵션에 대해 특정 값(value)을 지정. 애플리케이션 내부 로직에 따라 value로 올 수 있는 타입은 달라짐
  - `-silent`: 불리언 옵션값인 silent 옵션값

- 위치 인수(positional argument): arg1, arg2 등.
  - 순서대로 인수를 전달하는 방법으로, 순서가 달라지면 의미가 달라지는 방식.
  - 비교) 키워드 인수: 키=값 쌍으로 인수 전달
  - 위치 인수의 데이터 타입과 그에 대한 처리는 애플리케이션 로직에 달려있음.

#### 옵션과 위치 인수의 위치

flag 패키지는 위치 인수 파싱을 시작하면 -, --로 시작하는 옵션은 처리하지 않음! 때문에 **위치 인수는 모든 옵션들 뒤에 작성**해야 함.

- `-h`: h옵션 true 전달
- `-n 1 Hello`: n옵션에 1 전달, 위치인수로 Hello 전달
- `Hello -n 1`: 위치인수로 Hello 전달. n옵션 무시

### FlagSet

`FlagSet` 객체: 커맨드라인 애플리케이션의 인수를 처리하기 위한 추상 객체.

`Parse()`: 옵션 검사를 진행하며 **지정된 flag 값을 사전에 설정해놓은 변수들에 할당**. 에러 발생시 NewFlagSet의 매개변수2에 설정한 방식으로 처리(에러 반환 혹은 프로그램 실행 종료).

`NArg()`: flag 옵션 파싱 이후에 주어진 **위치 인수의 개수** 반환.

별도로 `-help`, `-h` 옵션 명시하지 않아도 디폴트 처리 지원.

### 프로젝트 프로덕션 코드 발췌

`./application 2` 대신 `./application -n 2`처럼 `-n` 옵션으로 값을 넘기도록 수정.

- `NewFlagSet(애플리케이션명, fs.Parse() 실행 도중 예외 처리 방식 지정)`: FlagSet 객체 반환

  - `ContinueOnError`: Parse() 함수가 nil 이외의 에러를 반환하더라도 프로그램 계속 실행. 파싱 에러 직접 처리하려는 목적.
  - `ExitOnError`: 오류 발생시 프로그램 종료.
  - `PanicOnError`: 오류 발생시 panic() 호출하여 프로그램 종료. recover() 함수를 통해 프로그램 종료 전에 마무리 정리 작업(cleanup action) 수행 가능.

- `SetOutput(writer)`: FlagSet 객체의 진단 메시지, 출력 메시지 작성을 위한 writer 지정.

  - 기본적으로 표준 에러인 `os.Stderr`를 값으로 전달받음.
  - 유닛 테스트에 활용 가능.

- `IntVar(값을 저장할 메모리 주소, "옵션명", 옵션의 기본 값, "사용자에게 보여주는 옵션의 목적")`

  - `IntVar(&c.numTimes, "n", 0, "Number of times to greet")`
  - n옵션의 값을 `&c.numTimes`에 저장. 즉, 매개변수1은 전달받은 옵션의 값을 저장할 메모리 주소인 포인터.
  - 프로그램의 사용법 출력시 매개변수4의 문자열 제시됨.

- IntVar 이외에도 float, bool, string, 사용자 지정 타입에 대해서도 플래그 옵션 설정 가능.

```go
func parseArgs(w io.Writer, args []string) (config, error) {
	c := config{}
	// NewFlagSet: FlagSet 객체 생성하여 반환
	fs := flag.NewFlagSet("greeter", flag.ContinueOnError)
	// SetOutput: FlagSet 객체에 writer 지정
	fs.SetOutput(w)
	// IntVar: int 타입의 옵션 지정
	fs.IntVar(&c.numTimes, "n", 0, "Number of times to greet")
	// Parse: 전달받은 args 슬라이스의 요소들 파싱하며 flag 옵션들에 대해 검사. IntVar에 의해 n 옵션은 발견시 *c.numTimes에 값 저장
	err := fs.Parse(args)
	// ContinueOnError이므로 Parse 도중에 발생하는 에러 직접 처리 가능.
	if err != nil {
		return c, err
	}
	// 위치 인수가 지정된 경우 예외 반환
	if fs.NArg() != 0 {
		return c, errors.New("Positional arguments specified")
	}
	return c, nil
}
```

#### 실행 예시

정상동작

```
./application -n 2
Your name please? Press the Enter key when done.
John
Nice to meet you John
Nice to meet you John
```

`-h`, `-help` 옵션을 명시적으로 지정하지 않은 경우, main 함수의 에러 처리 로직에 의해 마지막 줄이 보임

```bash
./application -h
Usage of greeter:
  -n int
        Number of times to greet
flag: help requested
```

예외 처리1

```bash
 ./application -n
flag needs an argument: -n
Usage of greeter:
  -n int
        Number of times to greet
flag needs an argument: -n
```

예외 처리2. 오류가 두 번 발생하는 점 개선 필요.

```bash
./application -n asd
invalid value "asd" for flag -n: parse error
Usage of greeter:
  -n int
        Number of times to greet
invalid value "asd" for flag -n: parse error
```

### 연습 미션

- `-o` 옵션 추가: -o 옵션의 변수값으로 파일시스템 경로 지정하면 해당 경로에 HTML 파일 출력
  - 입력된 사용자명이 "Jane Clancy"인 경우, `<h1>Hello Jane Clancy</h1>`
- `html/template` 패키지 활용
