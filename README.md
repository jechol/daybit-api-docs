
이 저장소는 [Slate](https://github.com/lord/slate)를 포크한 비공개 저장소다.

### For Developers
개발자는 정적 사이트 제너레이터인 [Middleman](https://github.com/middleman/middleman)의 기본 컨셉을 이해해야 한다. [Slate](https://github.com/lord/slate)는 [Middleman](https://github.com/middleman/middleman) 위에 작성된 boilerplate code 이다.


### Prerequisites

You're going to need:

 - **Linux or macOS** — Windows may work, but is unsupported.
 - **Ruby, version 2.3.1 or newer**
 - **Bundler** — If Ruby is already installed, but the `bundle` command doesn't work, just run `gem install bundler` in a terminal.

### Getting Set up
```shell
$ git clone https://github.com/daybit-exchange/daybit-api-docs
$ cd daybit-api-docs
$ asdf local ruby 2.5.1
$ gem install bundler
$ bundler install
```

### Run a server on `localhost`
```shell
$ bundler exec middleman server
```

### Build
```shell
$ bundler exec middleman build --clean
```

