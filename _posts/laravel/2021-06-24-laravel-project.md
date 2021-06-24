---
title: "라라벨(Laravel) 프로젝트 만들기"
date: 2021-06-24 13:44:28 -0400
categories: Laravel
Tags:
- Laravel
- Composer
- Github
---

- 라라벨 인스톨러 사용

```
$ composer global require "laravel/installer"
$ laravel new projectName
```

- 라라벨 인스톨러 With 젯스트림(jetstream) 

```
$ laravel new projectName --jet
```

- 컴포저(composer) 사용

```
$ composer create-project laravel/laravel projectName
```

- 깃허브(github) 복제 및 라라벨 프로젝트  설정

```
$ git clone https://url projectName
$ cd projectName
$ composer install
```
