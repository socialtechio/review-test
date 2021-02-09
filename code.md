## Agenda
Гипотетический разработчик вашей команды прислал свой код на code-review. Он хорошо изучил стандарты кода в компании и следует им, а также проверил работоспособность решения вручную. 

## ToDo:
Провести code-review предложенного решения. Необходимо указать на моменты, которые могут привести к проблемам при эксплуатации и расширении функционала.
Дополнительное (необязательное) задание: предложить свою реализацию функционала

### Решаемая разработчиком задача
- Данные о пользователе и его досье хранятся в базе данных
- Необходимо реализовать endpoint, который вернет данные о пользователе и его документе-досье в json формате

### Уточнения
- Используется PHP версии 7.4. и гипотетический MVC фреймворк
- Код попадет в продакшен, в дальнейшем команде нужно будет его поддерживать и расширять

### Код
Код решения также доступен [в файле](src/tt-code.php)
```php
<?php

class ProfileController extends AbstractController
{
    public function load(Request $request): string
    {
        $repository = new UserRepository();
        if (!$user = $repository->getByName($request->get('name'))) {
            return '';
        }
        return json_encode(['id' => $user->id, 'name' => $user->name, 'dossier' => ['data' => $user->dossier->data]]);
    }
}

class Dossier
{
    public User $user;
    public string $name;
    public string $data;
    public string $createdDate;

    public function __construct(User $user, string $name, string $data, string $createdDate)
    {
        $this->user = $user;
        $this->name = $name;
        $this->data = $data;
        $this->createdDate = $createdDate;
    }

    public function getDossierWasCorrected() {
        return (bool)strpos($this->name, 'CORRECTED');
    }
}

class User
{
    public int $id;
    public ?Dossier $dossier = null;
    public string $name;

    public function __construct(int $id, Dossier $dossier = null, string $name)
    {
        $this->id = $id;
        $this->dossier = $dossier;
        $this->name = $name;
    }
}

class UserRepository
{
    public function get(int $id): ?User
    {
        try {
            $db = Database::getInstance();
            $row = $db->query('SELECT * FROM users WHERE id= "' . $id . '" LIMIT 1');
            if (!count($row)) {
                return null;
            }

            $user = new User($row['id'], null, $row['name']);

            $docRow = $db->query('SELECT * FROM dossier WHERE user_id= "' . $id . '" LIMIT 1');
            if (!count($docRow)) {
                $user->__construct($row['id'], new Dossier($docRow['name'], $user), $row['name']);
            }
        } catch (\Throwable $exception) {
            return null;
        }

        return $user;
    }

    public function getByName(string $name): ?User
    {
        $db = Database::getInstance();
        if (!$row = $db->query('SELECT * FROM users WHERE name= "' . $name . '" LIMIT 1')) {
            return null;
        }

        return $this->get($row['id']);
    }

    public function getIds(int ...$ids): array
    {
        $users = [];
        foreach ($ids as $id) {
            $users[] = $this->get($id);
        }
        return $users;
    }
}

```
