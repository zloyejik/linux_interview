# Проект sema_aldpro_33

  

## Getting files

  

### Установить Git

Astra Linux:

```

sudo apt install git

```

Red OS:

```

dnf install git

```

## Prepare for Upload Files

  

### Настроить ssh

1) Создать файл ~/.ssh/gitlab_croc_ru

```

ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f $HOME/.ssh/gitlab_ald_33

```

  

2) Создать файл ~/.ssh/config

Со следующим содержимым:

```

Host gitlab.sema.loc

    PreferredAuthentications publickey

    IdentityFile ~/.ssh/gitlab__ald_33

```

  

3) Перейти по ссылке https://gitlab.croc.ru/-/user_settings/ssh_keys

Скопировать содержимое файла $HOME/.ssh/gitlab__ald_33.pub

Нажать "Add new key" и заполнить необходимые данные

  

### Скачать проект

```

git clone git@gitlab.croc.ru:croc_dit/GR_sys_ing/personal-groups/smaslennikov/astra/ald_pro/aldpro__ald_33.git

```

  

### Prepare for Merge Requst "sema"

  

```

cd existing_repo

git checkout -b sema_aldpro_33

git add --all

git commit -m "my comment"

git push -uf origin sema_aldpro_33

```

  

## Add your files

  

- [ ] [Create](https://docs.gitlab.com/ee/user/project/repository/web_editor.html#create-a-file) or [upload](https://docs.gitlab.com/ee/user/project/repository/web_editor.html#upload-a-file) files

- [ ] [Add files using the command line](https://docs.gitlab.com/ee/gitlab-basics/add-file.html#add-a-file-using-the-command-line) or push an existing Git repository with the following command:

  

```

cd existing_repo

git remote add origin git@gitlab.sema.loc:astra/ald_pro/sema_aldpro_33.git

git branch -M main

git push -uf origin main

```

