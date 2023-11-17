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

<br>

## Custom Ordering in Categories Tab
- `_layouts/categories.html` 파일을 생성하여 <https://seungriyou.github.io/categories/> 탭에서의 **최상위 카테고리 정렬 순서**를 커스터마이징 했다.

  > 원본 코드에서는 다음과 같이 카테고리 이름의 사전식 순서로 정렬하였다.
  > 
  > ```html
  > {% assign sort_categories = site.categories | sort %}
  > ```

- 해당 html 파일의 원본 파일은 [jekyll-theme-chirpy의 원본 `categories.html` 파일](https://github.com/cotes2020/jekyll-theme-chirpy/blob/master/_layouts/categories.html)에서 확인할 수 있다.

- post 작성 시 새로운 카테고리를 만들 것이라면, `_layouts/categories.html` 파일에 다음과 같이 `custom_order` 내에서 적절한 위치에 `, `와 함께 추가해주면 된다. 순서를 수정하고 싶을 때도 마찬가지로 `custom_order`를 수정한다.

  ```html
  <!-- [START] add custom order of categories in `CATEGORIES` tab -->
  <!-- ref: https://twpower.github.io/228-make-array-and-add-element-in-jekyll-liquid-en -->
  {% assign custom_order = 'Python, Dev-Log, MLOps, Problem Solving, Computer Science, Deep Learning, Experience, Daily-Log' | split: ', ' %}
  {% assign sort_categories = '' | split: ',' %}
  {% for co in custom_order %}
    {% for category in site.categories %}
      {% if category[0] == co %}
        {% assign sort_categories = sort_categories | push: category %}
      {% endif %}
    {% endfor %}
  {% endfor %}
  <!-- [END] add custom order of categories in `CATEGORIES` tab -->
  ```

- Jekyll liquid 문법에서 array를 생성하는 방법은 [Make array and add element in liquid](https://twpower.github.io/228-make-array-and-add-element-in-jekyll-liquid-en)를 참고했다.

<br>

## Small Tips for Posting
### 1. 이미지 크기 설정
```markdown
![example-image](/assets/img/posts/category/subcategory/image.png){: style="max-width: 70%"}
```

### 2. 텍스트 색상 설정
HTML color name은 [링크](https://htmlcolorcodes.com/color-names/)에서 확인하기!

```markdown
<span style="color: red">this is **red**</span>
```

### 3. Math: Curly Bracket
> ref: <https://github.com/orgs/community/discussions/16993#discussioncomment-4056560>

curly bracket을 표현하기 위해 `\{`, `\}`를 사용하면 출력이 안 된다. 대신, `\lbrace`와 `\rbrace`를 사용해야 한다.

- `{}` 출력 안 되는 예시
  ```markdown
  $\mathbf s^0=\{s^0(j)\in [n]\}_{j \in [m]}$
  ```

- `{}` 출력 되는 예시
  ```markdown
  $\mathbf s^0=\lbrace s^0(j)\in [n]\rbrace_{j \in [m]}$
  ```
