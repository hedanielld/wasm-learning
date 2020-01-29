# The hello example

In this example, we setup a `string` hello world WASM demo and demonstrate interaction with the browser's JS host. Since the `string` type is NOT natively supported by WASM, we use the `wasm-pack` and `wasm-bindgen` utilities to generate corresponding call stub objects in JS.

## Set up

```
$ sudo apt-get update
$ sudo apt-get -y upgrade

$ sudo apt-get -y install apache2
$ sudo chown -R $USER:$USER /var/www/html
$ sudo systemctl start apache2

$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
$ source $HOME/.cargo/env
$ rustup target add wasm32-wasi
$ rustup override set nightly
$ rustup target add wasm32-wasi --toolchain nightly

$ curl https://rustwasm.github.io/wasm-pack/installer/init.sh -sSf | sh
```


## Create new project

```
$ cargo new --lib hello
$ cd hello
```

## Change the hello/Cargo.toml file

```
[lib]
name = "hello_lib"
path = "src/lib.rs"
crate-type =["cdylib"]

[dependencies]
wasm-bindgen = "0.2.50"
```

## Write Rust code in src/lib.rs

```
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn say(s: String) -> String {
  let r = String::from("hello ");
  return r + &s;
}
```

## Build the WASM bytecode

```
$ wasm-pack build --target web
```

## Create a new HTML folder

```
$ mkdir html
$ cp pkg/hello_lib_bg.wasm html/
$ cp pkg/hello_lib.js html/
```

## Create a html/index.html file

```
<html>
  <head>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css" integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T" crossorigin="anonymous" />
    <script type="module">
      import init, { say } from './hello_lib.js';
      async function run() {
        await init();
        // const result = say("Michael");
        // console.log(result);
        var buttonOne = document.getElementById('buttonOne');
        buttonOne.addEventListener('click', function() {
          var input = $("#nameInput").val();
          alert(say(input));
        }, false);
      }
      run();
    </script>
  </head>
  <body>
    <div class="row">
      <div class="col-sm-4"></div>
      <div class="col-sm-4">
        <b>Wasm - Say hello</b>
      </div>
      <div class="col-sm-4"></div>
    </div>
    <hr />
    <div class="row">
      <div class="col-sm-2"></div>
      <div class="col-sm-4">What is your name?</div>
      <div class="col-sm-4"> Click the button</div>
      <div class="col-sm-2"></div>
    </div>
    <div class="row">
      <div class="col-sm-2"></div>
      <div class="col-sm-4">
        <input type="text" id="nameInput" placeholder="1", value="1">
      </div>
      <div class="col-sm-4">
        <button class="bg-light" id="buttonOne">Say hello</button>
      </div>
      <div class="col-sm-2"></div>
    </div>
  </body>
  <script
    src="https://code.jquery.com/jquery-3.4.1.js"
    integrity="sha256-WpOohJOqMqqyKL9FccASB9O0KwACQJpFTUBLTYOVvVU="
    crossorigin="anonymous"></script>
</html>
```

## Deploy the static files to Apache

```
$ cp html/* /var/www/html/
```

## Test

```
http://100.24.46.159/
```