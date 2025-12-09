### 1. Класс `Router`

```php
class Router
{
    private array $routes = [];

    public function add(string $method,
     string $path,
     array $controller, 
     array $ctorParams = [])
    {
        $this->routes[] = [
            'path' => $this->normalizePath($path),
            'method' => strtoupper($method),
            'controller' => $controller,
            'ctorParams' => $ctorParams,
        ];
    }

    private function normalizePath(string $path): string
    {
        $path = trim($path, '/');
        return "/{$path}/";
    }

    public function dispatch(string $requestPath, ?string $requestMethod = null)
    {
        $requestPath = $this->normalizePath($requestPath);
        $requestMethod = strtoupper($requestMethod ?? $_SERVER['REQUEST_METHOD'] ?? 'GET');

        foreach ($this->routes as $route) {
            // Создаём регэксп: {id} → ([^/]+)
            $pattern = preg_replace('/{[^}]+}/', '([^/]+)', $route['path']);
            
            if (!preg_match("#^{$pattern}$#", $requestPath, $matches)) {
                continue;
            }

            if ($route['method'] !== $requestMethod) {
                continue;
            }

            // Извлекаем имена параметров из шаблона
            preg_match_all('/{([^}]+)}/', $route['path'], $paramNames);
            $paramValues = array_slice($matches, 1); // значения из preg_match

            [$className, $action] = $route['controller'];

            // Создаём контроллер с параметрами конструктора
            $controller = new $className($route['ctorParams']);

            // Вызываем метод контроллера, передавая параметры как аргументы
            $reflection = new ReflectionMethod($className, $action);
            $reflection->invokeArgs($controller, $paramValues);

            return; // нашли и выполнили — выходим
        }

        // Маршрут не найден
        http_response_code(404);
        echo json_encode(['error' => 'Not Found']);
    }
}
```

---

### 2. Пример контроллера `NewsController`

```php
class NewsController
{
    private $repository;
    private $countPerPage;

    public function __construct(array $config)
    {
        $this->repository = $config['repository'] ?? null;
        $this->countPerPage = $config['countPerPage'] ?? 10;

        if (!$this->repository) {
            throw new InvalidArgumentException('Repository is required');
        }
    }

    // actionIndex() без аргументов → страница 1
    // actionIndex(2) → страница 2
    public function actionIndex(?int $page = 1)
    {
        if ($page < 1) {
            http_response_code(400);
            echo json_encode(['error' => 'Page must be >= 1']);
            return;
        }

        $offset = ($page - 1) * $this->countPerPage;
        $news = $this->repository->findAll($this->countPerPage, $offset);

        echo json_encode([
            'data' => $news,
            'page' => $page,
            'per_page' => $this->countPerPage
        ]);
    }

    // actionShow(1) → новость с id=1
    public function actionShow(int $id)
    {
        if ($id <= 0) {
            http_response_code(400);
            echo json_encode(['error' => 'Invalid ID']);
            return;
        }

        $newsItem = $this->repository->findById($id);
        if (!$newsItem) {
            http_response_code(404);
            echo json_encode(['error' => 'News not found']);
            return;
        }

        echo json_encode(['data' => $newsItem]);
    }
}
```

---

### 3. Настройка маршрутов

```php
$router = new Router();

// /news/ → actionIndex() (страница 1)
$router->add('GET', '/news/', [NewsController::class, 'actionIndex'], [
    'repository' => $newsRepository,
    'countPerPage' => 10
]);

// /news/page-2/ → actionIndex(2)
$router->add('GET', '/news/page-{page}/', [NewsController::class, 'actionIndex'], [
    'repository' => $newsRepository,
    'countPerPage' => 10
]);

// /news/show-1/ → actionShow(1)
$router->add('GET', '/news/show-{id}/', [NewsController::class, 'actionShow'], [
    'repository' => $newsRepository
]);

// Или классический REST-стиль: /news/1/
$router->add('GET', '/news/{id}/', [NewsController::class, 'actionShow'], [
    'repository' => $newsRepository
]);
```

---

### 4. Как это работает

1. **Шаблон маршрута**  
   - `{page}` в `/news/page-{page}/` заменяется на `([^/]+)` в регэкспе.
   - При совпадении URL значения захватываются и передаются в метод.

2. **Передача параметров**  
   - Значения из URL (`{page}`, `{id}`) передаются в метод **как отдельные аргументы**, а не массивом.
   - Для `actionIndex(?int $page = 1)`:  
     - `/news/` → `$page = 1` (значение по умолчанию);  
     - `/news/page-3/` → `$page = 3`.

3. **Конструктор контроллера**  
   - Параметры из `ctorParams` передаются в `__construct()` при создании экземпляра.

4. **HTTP-метод**  
   - Проверяется соответствие метода запроса (GET/POST и т. п.).

---

### 5. Примеры вызовов

| URL | Вызов метода |
|---|---|
| `/news/` | `newsController->actionIndex(1)` |
| `/news/page-5/` | `newsController->actionIndex(5)` |
| `/news/123/` | `newsController->actionShow(123)` |

---

### 6. Требования к среде

- PHP 8.0+ (для типизации аргументов вроде `?int $page`).
- Репозиторий (`$newsRepository`) должен иметь методы:  
  - `findAll(int $limit, int $offset): array`  
  - `findById(int $id): ?array`

---

**Итог:**  
- Маршрутизатор автоматически подставляет параметры из URL в аргументы методов.  
- Сохраняется ваш стиль именования действий (`actionIndex`, `actionShow`).  
- Гибкая настройка через шаблоны URL.