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

## Homepage

```rs
// src/pages/home.rs
use yew::prelude::*;
struct Product {
    id: i32,
    name: String,
    description: String,
    image: String,
    price: f64,
}

struct State {
    products: Vec<Product>,
}

pub struct Home {
    state: State,
}
impl Component for Home {
    type Message = ();
    type Properties = ();

    fn create(_: Self::Properties, _: ComponentLink<Self>) -> Self {
        let products: Vec<Product> = vec![
            Product {
                id: 1,
                name: "아롱".to_string(),
                description: "An apple a day keeps the doctor away".to_string(),
                image: "/products/고양이1.jpeg".to_string(),
                price: 3.65,
            },
            Product {
                id: 2,
                name: "호롱".to_string(),
                description: "An old banana leaf was once young and green".to_string(),
                image: "/products/고양이2.jpeg".to_string(),
                price: 7.99,
            },
        ];
        Self {
            state: State { products },
        }
    }

    fn update(&mut self, _: Self::Message) -> ShouldRender {
        true
    }

    fn change(&mut self, _: Self::Properties) -> ShouldRender {
        true
    }

    fn view(&self) -> Html {
        let products: Vec<Html> = self
            .state
            .products
            .iter()
            .map(|product: &Product| {
                html! {
                  <div>
                    <img src={&product.image}/>
                    <div>{&product.name}</div>
                    <div>{"$"}{&product.price}</div>
                  </div>
                }
            })
            .collect();

        html! { <span>{products}</span> }
    }
}

```

```rs
//lib
mod pages;

use pages::Home;
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
    App::<Home>::new().mount_to_body();
}

```

- create생명주기 메서드는 컴포넌트가 생성될떄 호출되며 여기서 초기 상태를 설정
- view메서드의 생명주기 메서드는 컴포넌트가 렌더된 뒤 발생한다.
- 위코드에서는 상품카드를 생성하기 위해 products를 반복햇다
- React에서의 view는 render,html!은 JSX

## 장바구니 기능

## 데이터 가져오기

```json
[
  {
    "id": 1,
    "name": "Apple",
    "description": "An apple a day keeps the doctor away",
    "image": "/products/apple.png",
    "price": 3.65
  },
  {
    "id": 2,
    "name": "Banana",
    "description": "An old banana leaf was once young and green",
    "image": "/products/banana.png",
    "price": 7.99
  }
]
```

```rs
[package]
  name = "rustmart"
  version = "0.1.0"
  authors = ["sheshbabu <sheshbabu@gmail.com>"]
  edition = "2018"

  [lib]
  crate-type = ["cdylib", "rlib"]

  [dependencies]
  yew = "0.17"
  wasm-bindgen = "0.2"
  anyhow = "1.0.32"
  serde = { version = "1.0", features = ["derive"] }
```

- create메서드에서 빈 배열로 하드코딩 되어 있었던 상품리스트 교체
- Msg::GetProducts메세지를 보내 update에서 api모듈에 위칲한 get_products메서드 호출 반환된 FetchTask는 task에 저장
- 요청이 성공하면 Msg::GetProductsSuccess메세지가 상품 리스트와 함께 호출되며, 실패할 경우 에러와 함께 Msg::getProductsError가 호출
- 두 메세지는 products와 get_products_error 필드를 각각 설정
- 요청이 완료된 후 get_products_loaded 상태 또한 true로 변경

## 재사용 컴포넌트 분리
