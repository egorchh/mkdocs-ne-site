# Задание 2. Кастомизация статического сайта (шаблонизация, сборка статики: HTML, CSS, JS)

На основе результатов выполнения предыдущей работы, кастомизируйте свой собственный сайт, разобравшись в том как возможно использовать свою собственную тему (шаблон для сайта) и создать свой шаблон (на занятии был продемонстрирован пример создания темы и шаблонизации с помощью Jinja2 для mkdocs), выполните сборку сайта с помощью GitHub Actions. 

## Задание: 

1. Создать собственную тему на основе HTML, CSS, JS с использованием или без использования CSS-библиотек таких как Bootstrap, Bulma, фреймворков (например, [Tailwind](https://tailwindcss.com/)), JS-библиотек для разработки фронтэнда (например, React). 

2. На основе [репозитория](https://github.com/nzhukov/deploy-ghactions) разработать пайплайн или набор пайплайнов (yml-файл) для тестирования и  сборки статики (HTML, CSS, JS) — фронтэнда сайта, а затем построения собственно самого сайта (интеграции контента в разметке Markdown в шаблон сайта) и деплоя его на GitHub Pages.

3. Необходимо учесть, что HTML файлы должны валидироваться на корректность, минифицироваться, должна быть предусмотрена сборка с помощью PostCSS (см. [репозиторий](https://github.com/nzhukov/deploy-ghactions)).

4. Дополнительным этапом (**необязательным**) реализуйте улучшение типографики содержимого сайта (например, используя этот [инструмент](https://github.com/typograf/typograf)). 

Требования: 

1. Минимальные требования к теме: наличие кастомного header, footer, а также стилизованной страницы для одной из секций (например, главной страницы). 

2. Обязательно добавьте метаданные сайта (название, описание, автор) (тег meta).

3. Сайт должен быть доступен по адресу GitHub Pages. Убедитесь, что все страницы и стили корректно отображаются.

## Отчет:

**[GitHub Pages](https://cavenikk.github.io/ITMO_web_python/)**

1. В корне проекта создал директорию theme, в которой разместил файлы `main.html`, `styles.css` и `script.js`.

2. Наполнил контентом `main.html`, написал стили в `styles.css`. В `script.js` поставил единственный `console.log` для проверки работы JavaScript. 

3. Указал в `mkdocs.yml` тему и пути до стилей и скрипта.

4. Инициализировал проект `Node`, установил `PostCSS` и `HTML Minifier`.

5. Указал в `package.json` скрипты, выполняющие минификацию HTML и CSS, а также скрипт `ci`, объединящий эти скрипты.

6. В корне проекта создал директории `.github` и `.github/workflows`. В последней создал файл `deploy.yml`, в котором указал все необходимые действия, а именно:

    - Установка Python 3.11
    - Установка mkdocs
    - Установка Node.js v20
    - Установка зависимостей npm
    - Билд статики `mkdocs build`
    - Запуск CI-скрипта `npm run ci`
    - Пуш в ветку `github-pages`

7. В настройках репозитория дал права на чтение токена для GitHub Actions.

8. Закоммитил и запушил изменения, и после окончания пайплайнов получил **[ссылку](https://cavenikk.github.io/ITMO_web_python/)**.

## Содержание Github Actions:

```yml
name: Deploy MkDocs Adidas Site

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.11'

      - name: Install Python dependencies
        run: |
          pip install mkdocs

      - name: Set up Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '20'

      - name: Install npm dependencies
        run: npm install

      - name: Build MkDocs site
        run: mkdocs build

      - name: HTML Validation
        run: |
          pip install html5validator
          html5validator --root site

      - name: Minify HTML
        run: |
          pip install htmlmin
          find site/ -name "*.html" -exec htmlmin --remove-comments --remove-empty-space {} -o {} \;

      - name: PostCSS processing
        run: |
          npm install postcss postcss-cli autoprefixer
          npx postcss site/styles.css --use postcss-import --use postcss-csso --no-map -o site/styles.css

      - name: Run CI Scripts
        run: npm run ci

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./site
          publish_branch: github-pages