## Отчет:

**[GitHub Pages](https://cavenikk.github.io/ITMO_web_python/)**

1. В корне проекта создал директорию theme, в которой разместил файлы `main.html`, `styles.css` и `script.js`.

2. Наполнил контентом `main.html`, написал стили в `styles.css`. В `script.js` поставил единственный `console.log` для проверки работы JavaScript. 

3. Указал в `mkdocs.yml` тему и пути до стилей и скрипта.

4. В корне проекта создал директории `.github` и `.github/workflows`. В последней создал файл `deploy.yml`, в котором указал все необходимые действия, а именно:

    - Установка Python 3.11
    - Установка mkdocs
    - Установка Node.js v20
    - Установка зависимостей npm
    - Билд статики `mkdocs build`
    - Валидация html
    - Минификация html
    - Обработка через postcss
    - Пуш в ветку `github-pages`

5. В настройках репозитория дал права на чтение токена для GitHub Actions.

6. Закоммитил и запушил изменения, и после окончания пайплайнов получил **[ссылку](https://cavenikk.github.io/ITMO_web_python/)**.

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