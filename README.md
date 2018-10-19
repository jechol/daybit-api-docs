# DAYBIT-API-DOCS
This is [Documents for DAYBIT APIs](https://docs.daybit.com/) and [Pydaybit](https://github.com/daybit-exchange/pydaybit).

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

## Document List

### DAYBIT API DOC 
> [ENGLISH](source/localizable/index.html.md)   
> [KOREAN](source/localizable/index.kr.html.md)

### PYDAYBIT DOC:
> [ENGLISH](source/includes/_pydaybit.md)   
> [KOREAN](source/includes/_pydaybit.kr.md)
