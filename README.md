# Tic-Tac-Toe Backend

Серверная реализация игры «Крестики-нолики» на Java/Spring с JWT-авторизацией, историей игр и таблицей лидеров.

## О проекте

Проект - развитие backend-части игры из предыдущей недели (T04). В этой версии Basic Auth заменён на **JWT-авторизацию** (access + refresh токены), добавлена **история завершённых игр** и **таблица лидеров** с рейтингом игроков по соотношению побед.

## Стек

- Java, Spring Boot (Web, Security, Data JPA)
- PostgreSQL + Hibernate
- JWT (`io.jsonwebtoken`)
- `CrudRepository`, `GrantedAuthority`, `SecurityContextHolder`
- `GenericFilterBean`, `SecurityFilterChain`
- UUID, Git/GitLab

## Что реализовано

### 1. JWT-авторизация
- Роль `USER` (enum) с поддержкой на всех слоях; `GrantedAuthority` для доменной модели роли; список ролей у пользователя.
- Модели `JwtRequest` (login, password), `JwtResponse` (type, accessToken, refreshToken), `RefreshJwtRequest` (refreshToken).
- `JwtProvider`: генерация accessToken (claims: UUID, роль) и refreshToken (claims: UUID), валидация обоих, получение claims.
- `JwtAuthentication` (расширяет `Authentication`): `getAuthorities()` → роли, `getPrincipal()`/`getName()` → UUID, отдельное поле статуса.
- `JwtUtil`: создание `JwtAuthentication` из claims.
- Сервис авторизации: вход по `JwtRequest` - `JwtResponse`, обновление access/refresh токенов, получение `JwtAuthentication`.
- Контроллер: эндпоинты авторизации, обновления accessToken и refreshToken.
- `AuthFilter`: извлечение `Bearer {accessToken}`, валидация, получение claims, сборка `JwtAuthentication`, установка в `SecurityContextHolder`.
- Эндпоинт получения информации о пользователе из accessToken.
- В `SecurityFilterChain` открыт без авторизации только эндпоинт обновления accessToken.

### 2. История игр
- Дата создания игры в модели.
- Запрос для получения всех завершённых игр по UUID пользователя (победа игрока или ничья).
- Метод сервиса и эндпоинт получения завершённых игр по accessToken (только для аутентифицированных).

### 3. Таблица лидеров
- Модель с UUID пользователя и win ratio.
- Запрос: соотношение побед к (поражения + ничьи), сортировка по убыванию, топ-N.
- Метод сервиса и эндпоинт топ-N игроков (UUID, login, win ratio), только для аутентифицированных.
