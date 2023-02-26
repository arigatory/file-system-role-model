# file-system-role-model

## Основные идеи взаимодействия между микросервисами Users, AccessControl и FileSystem:

* Микросервис Users отвечает за управление учетными записями пользователей, аутентификацию и авторизацию. Он предоставляет API для создания, обновления и удаления учетных записей пользователей, а также для аутентификации пользователей и генерации токенов доступа.

* Микросервис AccessControl отвечает за управление правилами контроля доступа к файловой системе. Он предоставляет API для создания, обновления и удаления правил контроля доступа (разрешений) для каждого элемента файловой системы. Он также предоставляет API для проверки того, имеет ли пользователь необходимый уровень доступа к определенному элементу файловой системы.

* Микросервис FileSystem отвечает за управление самой файловой системой. Он предоставляет API для создания, обновления и удаления элементов файловой системы (файлов и папок). Он также предоставляет API для получения элемента файловой системы по его ID.

* Когда пользователь хочет выполнить операцию с элементом файловой системы (например, прочитать файл), он отправляет запрос к микросервису AccessControl с его токеном доступа и ID элемента файловой системы.

* Микросервис AccessControl проверяет токен доступа пользователя и извлекает роли пользователя из микросервиса Users. Затем он проверяет, имеют ли какие-либо роли разрешения, позволяющие запрашиваемую операцию с элементом файловой системы.

* Если пользователь авторизован для выполнения операции, микросервис AccessControl отправляет запрос к микросервису FileSystem для получения элемента файловой системы по его ID.

* Микросервис FileSystem получает элемент файловой системы и возвращает его микросервису AccessControl.

* Микросервис AccessControl затем возвращает элемент файловой системы пользователю. Если пользователь не авторизован для выполнения операции, микросервис AccessControl возвращает сообщение об ошибке.

## Пример
Один из примеров, как можно моделировать файловую систему, если хотим хранить доступы в отдельном микросервисе

Если нужно разделить класс Permission от класса FileSystemItem на разные микросервисы, можно использовать такой подход, как domain-driven design (DDD), чтобы определить четкие границы между микросервисами и управлять взаимодействием между ними.

Вот пример того, как можно написать классы на C#, чтобы поддерживать этот подход:

Во-первых, мы можем создать микросервис FileSystem, который управляет файловой системой, включая классы FileSystemItem, File и Folder. Этот микросервис будет предоставлять API для взаимодействия с файловой системой, такие как создание, чтение, обновление и удаление файлов и папок.

``` c#
public class FileSystemItem
{
    public string Id { get; set; }
    public string Name { get; set; }
    public DateTime CreationDate { get; set; }
    public DateTime LastModifiedDate { get; set; }
    public FileSystemItemType ItemType { get; set; }
    public string ParentId { get; set; }
    public List<FileSystemItem> Children { get; set; }
}

public enum FileSystemItemType
{
    File,
    Folder
}

public class File : FileSystemItem
{
    public string Content { get; set; }
    public string Extension { get; set; }
    public int Size { get; set; }
}

public class Folder : FileSystemItem
{
    public List<FileSystemItem> Items { get { return Children; } }
}

public interface IFileSystemService
{
    FileSystemItem CreateFile(string name, string parentId, string content);
    FileSystemItem CreateFolder(string name, string parentId);
    FileSystemItem GetFileSystemItem(string id);
    List<FileSystemItem> GetChildren(string id);
    void UpdateFileSystemItem(FileSystemItem item);
    void DeleteFileSystemItem(string id);
}
```

Затем мы можем создать микросервис AccessControl, который управляет разрешениями и контролем доступа. Этот микросервис будет предоставлять API для управления ролями и разрешениями, и он будет взаимодействовать с микросервисом FileSystem для применения этих разрешений к файловой системе.

``` c#
public class Role
{
    public string Id { get; set; }
    public string Name { get; set; }
}

public class Permission
{
    public string Id { get; set; }
    public string FileSystemItemId { get; set; }
    public string RoleId { get; set; }
    public AccessLevel AccessLevel { get; set; }
}

public enum AccessLevel
{
    None,
    Read,
    Write,
    Execute
}

public interface IAccessControlService
{
    Role CreateRole(string name);
    Permission GrantPermission(string fileSystemItemId, string roleId, AccessLevel accessLevel);
    void RevokePermission(string permissionId);
    bool CheckAccess(string fileSystemItemId, string roleId, AccessLevel accessLevel);
}
```

В этом примере класс Permission был спроектирован так, что включает FileSystemItemId и RoleId вместо ссылки на классы FileSystemItem и Role. Интерфейс IAccessControlService определяет методы создания ролей, предоставления и отзыва разрешений, а также проверки доступа.

Чтобы использовать эти микросервисы вместе, микросервис AccessControl вызовет метод GetFileSystemItem микросервиса FileSystem, чтобы получить объект FileSystemItem, а затем использовать его свойство Id для установки свойства FileSystemItemId объекта Permission.

В целом, этот подход позволяет разделить задачи управления файловой системой и управления контролем доступа, а также определить четкие границы между микросервисами.
