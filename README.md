# Chirpy Starter [![Gem Version](https://img.shields.io/gem/v/jekyll-theme-chirpy)](https://rubygems.org/gems/jekyll-theme-chirpy) [![GitHub license](https://img.shields.io/github/license/cotes2020/chirpy-starter.svg?color=blue)][mit]

When installing the [**Chirpy**][chirpy] theme through [RubyGems.org][gem], Jekyll can only read files in the folders `/_data`, `/_layouts`, `/_includes`, `/_sass` and `/assets`, as well as a small part of options of the `/_config.yml` file from the theme's gem. If you have ever installed this theme gem, you can use the command `bundle info --path jekyll-theme-chirpy` to locate these files.

The Jekyll team claims that this is to leave the ball in the user’s court, but this also results in users not being able to enjoy the out-of-the-box experience when using feature-rich themes.

To fully use all the features of **Chirpy**, you need to copy the other critical files from the theme's gem to your Jekyll site. The following is a list of targets:

```shell
.
├── _config.yml
├── _plugins
├── _tabs
└── index.html
```

To save you time, and also in case you lose some files while copying, we extract those files/configurations of the latest version of the **Chirpy** theme and the [CD][CD] workflow to here, so that you can start writing in minutes.

## Prerequisites

Follow the instructions in the [Jekyll Docs](https://jekyllrb.com/docs/installation/) to complete the installation of `Ruby`, `RubyGems`, `Jekyll` and `Bundler`.

## Installation

[**Use this template**][use-template] to generate a brand new repository and name it `<GH_USERNAME>.github.io`, where `GH_USERNAME` represents your GitHub username.

Then clone it to your local machine and run:

```
$ bundle
```

## Usage

Please see the [theme's docs](https://github.com/cotes2020/jekyll-theme-chirpy#documentation).

## License

This work is published under [MIT][mit] License.

[gem]: https://rubygems.org/gems/jekyll-theme-chirpy
[chirpy]: https://github.com/cotes2020/jekyll-theme-chirpy/
[use-template]: https://github.com/cotes2020/chirpy-starter/generate
[CD]: https://en.wikipedia.org/wiki/Continuous_deployment
[mit]: https://github.com/cotes2020/chirpy-starter/blob/master/LICENSE

<br>

<hr>

<br>

# Personal Notes

## TODO
- [x] `_config.yml` - `google_site_verification`, `google_analytics`, `giscus` 정보 추가하기
- [ ] `a` tag area가 `img`를 벗어나는 문제
- [ ] image에서 `.left` / `.right` / `.normal` 등이 먹지 않는 문제

<br>

## How to Upgrade
> starter로 설치한 경우, 다음과 같이 하면 된다.

If you are using the theme gem (there will be gem "jekyll-theme-chirpy" in the Gemfile), editing the Gemfile and update the version number of the theme gem, for example:

```diff
- gem "jekyll-theme-chirpy", "~> 3.2"
+ gem "jekyll-theme-chirpy", "~> 4.0"
```
And then execute the following command:
```shell
bundle update jekyll-theme-chirpy
```
As the version upgrades, the critical files (for details, see the startup template) and configuration options will change. We can use the GitHub API to get the file changes in the version upgrade.

The URL format is as follows:
```
https://github.com/cotes2020/chirpy-starter/compare/<older_version>...<newer_version>
```
For instance, to upgrade from v4.0.0 to v5.0.0, visit:
https://github.com/cotes2020/chirpy-starter/compare/v4.0.0...v5.0.0

<br> 

> references
> - [https://github.com/cotes2020/jekyll-theme-chirpy/wiki/Upgrade-Guide](https://github.com/cotes2020/jekyll-theme-chirpy/wiki/Upgrade-Guide)
> - [https://chirpy.cotes.page/posts/getting-started/#upgrading](https://chirpy.cotes.page/posts/getting-started/#upgrading)


<br>

## jekyll, gem 관련 이슈
### `GemNotFound`
다음과 같은 에러가 발생할 때는 

```
... `materialize': Could not find sass-embedded-1.58.3 in any of the sources (Bundler::GemNotFound)
```

다음의 명령어로 `gem`을 정리한다.
```shell
bundler
```

### `jekyll s` not working
다음의 명령어로 실행한다.
```shell
bundle exec jekyll s
```
(23.10.27 추가) 요즘은 다음과 같이 실행해도 된다...!
```shell
jekyll s
```

### `rbenv`
```shell
rbenv versions
rbenv local
rbenv local 3.1.0
```

<br>

## Adding Custom CSS Settings
- `_etc` 내부의 sass 파일들은 기존에 배포된 chirpy theme의 파일이다. 해당 파일들을 참고하면 된다.

- 실제 custom 파일은 `_sass/custom` 폴더 밑에 작성한다. 

  - 새로 파일을 작성할 경우, `assets/css/style.scss`에 `@import` 해주면 된다.
  - 예외로, 포스팅 내 코드 블럭의 색상 팔레트 설정은 `_sass/colors`에 위치시켜 별다른 `@import` 작업 없이 & overriding 동작 없이 적용되도록 한다.

- `@mixin`을 override 하기 위해 삽질을 오래했는데, `_sass/custom/addon/commons.scss`, `_sass/custom/addon/syntax.scss` 파일을 참고하여 사용하면 override 동작도 수행할 수 있다.
