## Getting Set up

### Installation
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

