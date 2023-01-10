# yew_test

## yew

- WebAssembly(wasm)을 사용하면 자바스크립트 외의 언어로 작성된 코드를 브라우저에서 실행할 수 있다.

## 설치

```
$ cargo install wasm-pack          # Rust를 컴파일 해 Wasm과 JS Interop 코드를 생성
$ cargo install cargo-make         # 태스크 러너
$ cargo install simple-http-server # assets을 실행하는 Simple Server
```

## 프로젝트 생성

```
$ cargo new --lib 프로젝트이름 && cd 프로젝트이름
```

## Cargo.toml

```
[lib]
crate-type = ["cdylib", "rlib"]

[dependencies]
yew = "0.17"
wasm-bindgen = "0.2"
```

## Makefile.toml

- Makefile.toml도 생성해 다음 내용을 추가한다.

```
[tasks.build]
command = "wasm-pack"
args = ["build", "--dev", "--target", "web", "--out-name", "wasm", "--out-dir", "./static"]
watch = { ignore_pattern = "static/*" }

[tasks.serve]
command = "simple-http-server"
args = ["-i", "./static/", "-p", "3000", "--nocache", "--try-file", "./static/index.html"]
```

## build

```
cargo make build
```

## hello world

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>RustMart</title>
    <script type="module">
      import init from "/wasm.js";
      init();
    </script>
    <link rel="shortcut icon" href="#" />
  </head>
  <body></body>
</html>
```

## src/lib.rs

```rs
// src/lib.rs
use wasm_bindgen::prelude::*;
use yew::prelude::*;

struct Hello {}

impl Component for Hello {
    type Message = ();
    type Properties = ();

    fn create(_: Self::Properties, _: ComponentLink<Self>) -> Self {
        Self {}
    }

    fn update(&mut self, _: Self::Message) -> ShouldRender {
        true
    }

    fn change(&mut self, _: Self::Properties) -> ShouldRender {
        true
    }

    fn view(&self) -> Html {
        html! { <span>{"Hello World!"}</span> }
    }
}

#[wasm_bindgen(start)]
pub fn run_app() {
    App::<Hello>::new().mount_to_body();
}
```

- hello라는 컴포넌트 생성해 Hello world을 생성하는 작업을 한다.

## 실행

```
cargo make serve
```

- http://localhost:3000

## 컴포넌트

- props:해단 컴포넌트에서 자식 컴포넌트로 데이터나 콜백을 전달
- state:컴포넌트의 로컬 상태를 조작
- AppState:전역 상태를 조작

- 리렌더링

```
1.props의 변경
2.State의 변경
3.AppState의 변경
```
