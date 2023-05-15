## Agenda
Гіпотетичний розробник вашої команди надіслав свій код на code-review. Він добре вивчив стандарти коду в компанії і слідує їм, а також перевірив працездатність рішення вручну.

## ToDo:
Провести code-review запропонованого рішення. Необхідно вказати на моменти, які можуть призвести до проблем під час експлуатації та розширення функціонала.
Додаткове (необов'язкове) завдання: запропонувати свою реалізацію функціонала.

### Розв'язуване розробником завдання
- Дані про користувача та його досьє зберігаються у базі даних
- Необхідно реалізувати endpoint, який поверне дані про користувача та його документ-досьє у json форматі

### Уточнення
- використовується PHP версії 7.4. та гіпотетичний MVC фреймворк
- Код потрапить у продакшн, надалі команді потрібно буде його підтримувати та розширювати.

### Код
Код рішення також доступний [в файлі](src/tt-code.php)
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
            $row = $db->query('SELECT * FROM users WHERE id= ' . $id . ' LIMIT 1');
            if (!count($row)) {
                return null;
            }

            $user = new User($row['id'], null, $row['name']);

            $docRow = $db->query('SELECT * FROM dossier WHERE user_id= ' . $id . ' LIMIT 1');
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
