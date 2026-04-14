Три способа прокинуть данные в контейнер:

**Bind mount** — папка хоста монтируется в контейнер:
```bash
docker run -v /host/path:/container/path myapp

# docker-compose
volumes:
  - ./config:/app/config        # конфиг
  - ./data:/var/lib/postgresql  # данные БД
```

**Named volume** — Docker сам управляет хранилищем:
```bash
docker volume create pgdata
docker run -v pgdata:/var/lib/postgresql/data postgres

# docker-compose
volumes:
  - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:  # Docker создаёт и хранит в /var/lib/docker/volumes/
```

**tmpfs** — в памяти, исчезает при остановке:
```bash
docker run --tmpfs /tmp myapp
```

| Тип         | Персистентность | Кто управляет | Когда использовать              |
| :---------- | :-------------- | :------------ | :------------------------------ |
| Bind mount  | Да, на хосте    | Разработчик   | Dev: конфиги, код, hot reload   |
| Named volume| Да, в Docker    | Docker        | Prod: данные БД, файлы         |
| tmpfs       | Нет             | —             | Секреты, временные файлы       |