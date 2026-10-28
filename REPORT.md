# Отчёт по мидтерму

Студент: Dzholdosheva Nuraiym
Репозиторий: https://github.com/dzholdoshevan-del/infra-project

## Блок 1. Основы

### 1.1 Настройка и первый коммит
Команды: git config --global user.name / user.email / init.defaultBranch main, git clone git@github.com:dzholdoshevan-del/infra-project.git, создание .gitignore (.env, *.log, *.retry), git commit, git push -u origin main.
Вывод: [main (root-commit) fbf1326] chore: add .gitignore
Первый коммит содержит только .gitignore, репозиторий создан пустым.

### 1.2 Файлы проекта
Создал .env (DB_PASSWORD=secret123) и app.log. git status показывает только README.md, .env.example, ansible/, docker/ (.env и app.log не видны). Три отдельных коммита и push:

    1921ace feat(ansible): add playbook, inventory and config
    cb5de71 feat(docker): add Dockerfile and index page
    1221ff7 docs: add README and .env.example
    fbf1326 chore: add .gitignore

git ls-files не содержит .env и app.log. .gitignore скрыл секретные и служебные файлы.

### 1.3 Просмотр изменений
Дописал строку в README.md. Вывод git diff до git add: +Проект для практики: Docker и Ansible. После git add второй git diff пустой, git diff --staged показывает то же изменение. Коммит 3e7adcd.
Второй git diff пустой, потому что он сравнивает рабочую папку с индексом, а после git add они совпадают. git diff --staged сравнивает индекс с последним коммитом.
Примечание: при первой попытке русский текст записался в неверной кодировке, исправил отдельным коммитом 03d3cc8 (docs: fix README encoding), force-push не использовал.

## Блок 2. История и ветки

### 2.1 История
Команды: git log --oneline, git log --oneline --graph --all, git show HEAD~1.
HEAD~1 соответствует коммиту 03d3cc8 docs: fix README encoding.

### 2.2 Ветка и слияние
Команды: git switch -c feature/healthcheck, коммит строки HEALTHCHECK в docker/Dockerfile, git switch main, git merge feature/healthcheck, git branch -d feature/healthcheck.
Вывод merge: Updating 3e7adcd..3f749bd, Fast-forward. Ветка удалена, git branch показывает только main.
Позже обнаружил, что строка HEALTHCHECK склеилась с COPY, исправил коммитом 98ed114 fix(docker): put HEALTHCHECK on its own line.

### 2.3 Конфликт слияния
Ветка fix/app-port: app_port: 9090 (f90e677). В main: app_port: 8000 (e2d6630). git merge fix/app-port:

    CONFLICT (content): Merge conflict in ansible/group_vars/all.yml

Файл с маркерами:

    ---
    <<<<<<< HEAD
    app_port: 8000
    =======
    app_port: 9090
    >>>>>>> fix/app-port
    app_user: deploy

Выбрал 8000, удалил маркеры, git add и git commit (772864d merge: resolve app_port conflict). Итоговый файл: app_port: 8000, app_user: deploy. В git log --oneline --graph виден merge-коммит с развилкой.

## Блок 3. Удалённый репозиторий и Merge Request

### 3.1 Получение изменений
Отредактировал README.md в веб-интерфейсе GitHub (коммит Update README.md в main). Локально git pull:

    Updating 7a66498..3aa0684
    Fast-forward
     README.md | 1 +

git log --oneline: 3aa0684 Update README.md.

### 3.2 Merge Request
Ветка feature/ansible-curl, в ansible/playbook.yml добавлена задача установки curl. Коммит в формате Conventional Commits: feat(ansible): add curl installation task. git push -u origin feature/ansible-curl, Pull Request #2 в main с описанием, слит через веб-интерфейс (7a66498 Merge pull request #2). После git pull в playbook.yml есть curl (строки 12 и 14).

### 3.3 Уборка веток
Команды: git branch -d feature/ansible-curl, git push origin --delete feature/ansible-curl, git fetch --prune, git branch -a. Вывод:

    fix/app-port
  * main
    remotes/origin/HEAD -> origin/main
    remotes/origin/main

Ветки feature/ansible-curl больше нет ни локально, ни на сервере.

## Блок 4. Docker

Ветка feature/docker-page: страница docker/index.html с моим именем, EXPOSE 80 после COPY, HEALTHCHECK сохранён.

    docker build -t infra-web:1.0 docker/
     => [2/2] COPY index.html /usr/share/nginx/html/
     => => naming to docker.io/library/infra-web:1.0

    docker run -d --name infra-web -p 8080:80 infra-web:1.0
    curl.exe http://localhost:8080
    <h1>Infra project</h1>
    <p>Made by Dzholdosheva Nuraiym</p>

Контейнер удалён: docker rm -f infra-web. В README добавлен раздел Запуск в Docker с командами. Коммит 5ec3c57 feat(docker): add personal page and expose port, Pull Request #3 слит (aaee70b), EXPOSE 80 есть в main.

## Блок 5. Исправление ошибок

### 5.1 restore и amend
Дописал лишнюю строку в docker/Dockerfile (git status --short: M docker/Dockerfile), вернул командой git restore docker/Dockerfile (git status --short пуст).
Коммит с опечаткой и исправление до push:

    git commit -m "docs: updte readme"
    git commit --amend -m "docs: update readme"

В истории на сервере: 95eddda docs: update readme, коммита с updte нет.

### 5.2 revert
Удалил *.log из .gitignore: 810341e chore: remove *.log from gitignore, push. Затем:

    git revert HEAD --no-edit
    bfa670a Revert "chore: remove *.log from gitignore"

.gitignore снова содержит .env, *.log, *.retry.
Для уже отправленного коммита используют revert, потому что удаление коммита из истории потребует force-push и сломает копии репозитория у других людей. Revert добавляет новый коммит-отмену и сохраняет историю.

## Контрольные вопросы

1. Коммит это снимок состояния проекта с автором, сообщением и хэшем. Индекс (staging area) нужен, чтобы выбрать, какие именно изменения войдут в следующий коммит, и собирать коммиты по смыслу.
2. Ветка это подвижный указатель на коммит, отдельная линия разработки, в которой можно работать независимо от main.
3. HEAD это указатель на текущее положение: обычно на текущую ветку и её последний коммит.
4. git fetch только скачивает новые данные с сервера и не меняет мои файлы, а git pull это fetch плюс слияние в текущую ветку.
5. В .env лежат пароли и ключи. Если добавить его в репозиторий, их увидят все, у кого есть доступ, и они останутся в истории навсегда, даже после удаления файла.