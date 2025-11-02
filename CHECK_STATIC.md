# Команды для проверки статики на сервере

## Подключение:
```bash
ssh -i C:/vm_access/yc-shamil-nft yc-user@51.250.26.21
```

## Проверка статики:

### 1. Проверить что контейнеры запущены:
```bash
cd ~/taski
docker compose -f docker-compose.production.yml ps
```

### 2. Проверить содержимое volume static_volume:
```bash
docker run --rm -v taski_static_volume:/mnt alpine ls -la /mnt
```

### 3. Проверить что файлы есть в gateway:
```bash
docker compose -f docker-compose.production.yml exec gateway ls -la /staticfiles/
```

### 4. Проверить что frontend скопировал файлы:
```bash
docker compose -f docker-compose.production.yml exec gateway ls -la /staticfiles/ | head -20
docker compose -f docker-compose.production.yml exec gateway test -f /staticfiles/index.html && echo "index.html exists" || echo "index.html NOT FOUND"
```

### 5. Проверить что backend скопировал статику Django:
```bash
docker compose -f docker-compose.production.yml exec gateway ls -la /staticfiles/admin/ 2>/dev/null && echo "Django admin static exists" || echo "Django admin static NOT FOUND"
```

### 6. Проверить логи frontend (скопировал ли файлы):
```bash
docker compose -f docker-compose.production.yml logs frontend
```

### 7. Проверить логи backend (собрал ли статику):
```bash
docker compose -f docker-compose.production.yml logs backend | grep -i static
```

### 8. Вручную скопировать статику если нужно:
```bash
# Backend статика:
docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic --noinput
docker compose -f docker-compose.production.yml exec backend sh -c "cp -r /app/collected_static/. /backend_static/static/"

# Frontend статика:
docker compose -f docker-compose.production.yml exec frontend sh -c "cp -r /app/build/. /frontend_static/"
```

### 9. Проверить nginx конфиг внутри gateway:
```bash
docker compose -f docker-compose.production.yml exec gateway cat /etc/nginx/conf.d/default.conf
```

### 10. Проверить nginx доступ к статике:
```bash
docker compose -f docker-compose.production.yml exec gateway nginx -t
docker compose -f docker-compose.production.yml restart gateway
```

## Создание суперпользователя Django:

```bash
# Правильная команда (через python manage.py):
docker compose -f docker-compose.production.yml exec backend python manage.py createsuperuser

# Если нужен sudo:
sudo docker compose -f docker-compose.production.yml exec backend python manage.py createsuperuser

# Интерактивно введите:
# - Username (leave blank to use 'root'):
# - Email address:
# - Password:
# - Password (again):
```

## Возможные проблемы:

1. **Frontend не скопировал файлы** - нужно запустить frontend контейнер снова
2. **Backend не собрал статику** - нужно выполнить collectstatic и скопировать
3. **Volume пустой** - файлы не были скопированы
4. **Nginx неправильно настроен** - проблема с alias/root в конфиге
5. **Ошибка "createsuperuser not found"** - используйте `python manage.py createsuperuser`

