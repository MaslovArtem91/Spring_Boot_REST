# Spring_Boot_REST
# Задача Сервис авторизации

## Описание
Сегодня мы с вами реализуем сервис авторизации пользователей по логину и паролю. Но ключевым в этом задании будет то, как ваше приложение будет реагировать на ошибки, которые наш сервис будет выбрасывать в разных случаях.

Для работы необходимо подготовить несколько классов:

0. Создайте spring boot приложение и все классы контроллеры, сервисы и репозитории сделать бинами в вашем application context.

1. Запрос на разрешения будет приходить на контроллер:

```java
@RestController
public class AuthorizationController {
    AuthorizationService service;
    
    @GetMapping("/authorize")
    public List<Authorities> getAuthorities(@RequestParam("user") String user, @RequestParam("password") String password) {
        return service.getAuthorities(user, password);
    }
}
``` 

2. Класс-сервис, который будет обрабатывать введенные логин и пароль, выглядит следующим образом. 

```java
public class AuthorizationService {
    UserRepository userRepository;

    List<Authorities> getAuthorities(String user, String password) {
        if (isEmpty(user) || isEmpty(password)) {
            throw new InvalidCredentials("User name or password is empty");
        }
        List<Authorities> userAuthorities = userRepository.getUserAuthorities(user, password);
        if (isEmpty(userAuthorities)) {
            throw new UnauthorizedUser("Unknown user " + user);
        }
        return userAuthorities;
    }

    private boolean isEmpty(String str) {
        return str == null || str.isEmpty();
    }

    private boolean isEmpty(List<?> str) {
        return str == null || str.isEmpty();
    }
}
``` 
Он принимает в себя логин и пароль и возвращает разрешения для этого пользователя, если такой пользователь найден и данные валидны. Если присланные данные неверны, тогда выкидывается InvalidCredentials:

```java
public class InvalidCredentials extends RuntimeException {
    public InvalidCredentials(String msg) {
        super(msg);
    }
}
``` 

Если наш репозиторий не вернул никаких разрешений, либо вернул пустую коллекцию, тогда выкидывается ошибка UnauthorizedUser:

```java
public class UnauthorizedUser extends RuntimeException {
    public UnauthorizedUser(String msg) {
        super(msg);
    }
}
``` 

Enum с разрешениями выглядит следующим образом:

```java
public enum Authorities {
    READ, WRITE, DELETE
}
``` 

3. Код метод getUserAuthorities класс UserRepository, который возвращает либо разрешения, либо пустой массив, надо реализовать вам.

```java
public class UserRepository {
    public List<Authorities> getUserAuthorities(String user, String password) {
        return ...;
    }
}
``` 

Для проверки работоспособности можно из браузера сделать следующий запрос, заполнив `<ИМЯ_ЮЗЕРА>` и `<ПАРОЛЬ_ЮЗЕРА>` своими тестовыми данными: localhost:8080/authorize?user=<ИМЯ_ЮЗЕРА>&password=<ПАРОЛЬ_ЮЗЕРА>

4. Теперь, когда весь код у вас готов, то вам необходимо написать обработчики ошибок, которые выкидывает сервис `AuthorizationService`. Требования к ним такие:
     - На `InvalidCredentials` он должен обратно клиенту отсылать http статус с кодом 400 и телом в виде сообщения из exception'а
     - На `UnauthorizedUser` он должен обратно клиенту отсылать http статус с кодом 401 и телом в виде сообщения из exception'а и писать в консоль сообщение из exception'а
 

Задача Dockerfile
Давайте соберем наш первый докер образ на основе нашего приложения авторизации, которое мы писали во втором домашнем задании(возьем чисто серверное нашего приложение без html из прошлого задания). Для этого мы сначала напишем наш Dockerfile, а затем, для удобства, напишем манифест для docker-compose

Описание
Первым делом нам надо собрать jar архив с нашим spring boot приложением. Для этого в терминале в корне нашего проект выполните команду:
Для gradle: ./gradlew clean build (если пишет Permission denied тогда сначала выполните chmod +x ./gradlew)

Для maven: ./mvnw clean package (если пишет Permission denied тогда сначала выполните chmod +x ./mvnw)

Теперь можно начинать писать Dockerfile. Базовым образом возьмите openjdk:8-jdk-alpine и не забудьте открыть докеру порт(EXPOSE), на котором работает ваше приложение

Добавьте собранный jar в ваш образ(ADD). Если вы собирали с помощью maven, тогда jar будет лежать в папке target, а если gradle - в build/libs

Для удобства сборки образа и запуска контейнера нашего приложения, напишем docker-compose.yml. Контейнер назовите как вам больше нравится, а в его конфигурациях пропишите следующее:

добавим build: ./ который скажет docker-compose что надо сначала собрать образ для этого контейнера
добавим соответствие порта на хост машине и порта в контейнере для нашего приложения (аналог аргумента -p у команды docker run)
Два полученных файла добавьте в репозиторий вашего приложения и пришлите ссылка на него.
