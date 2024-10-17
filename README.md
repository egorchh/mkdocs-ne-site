# Мой сайт на GitHub Pages

## Ссылка на сайт:
https://egorchh.github.io/mkdocs-ne-site/

## Описание проекта

Этот проект представляет собой сайт, созданный с использованием MkDocs и кастомной темы на основе HTML, CSS и JS. Для сборки и деплоя использован GitHub Actions.

## Этапы выполнения

1. **Создание кастомной темы**:
    - Кастомизированы header и footer.
    - Стилизована главная страница.
    - Добавлены метаданные для сайта.

2. **Настройка GitHub Actions**:
    - Валидация и минификация HTML.
    - Сборка CSS с использованием PostCSS.
    - Автоматический деплой на GitHub Pages.

## Содержимое workflows/ci.yml:

```yaml
name: Build and Deploy yo!

on:
  push:
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
          python-version: '3.x'

      - name: Install dependencies
        run: |
          pip install mkdocs mkdocs-material

      - name: Build site
        run: mkdocs build --clean

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
          npx postcss overrides/main.css --use autoprefixer -o overrides/main.min.css

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./site
          publish_branch: gh-pages
