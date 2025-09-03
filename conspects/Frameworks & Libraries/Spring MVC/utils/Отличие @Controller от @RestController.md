## 1. `@Controller`

Это базовая аннотация Spring MVC. Говорит Spring-у: **этот класс — компонент веб-слоя, который обрабатывает HTTP-запросы**. Методы внутри такого контроллера по умолчанию возвращают **имя представления (view)**, которое потом ищется в `ViewResolver` (например, JSP, Thymeleaf и т.п.). Чтобы вернуть именно **данные (JSON, XML, строку)**, надо добавлять `@ResponseBody` к методу.

Про ищется в `ViewResolver`:
```java
@Controller
@RequestMapping("/home")
public class HomeController {

    @GetMapping
    public String index() {
        return "index"; 
    }
}
```

Здесь строка `"index"` — **это не сам текст "index"**, а **имя view-шаблона**.  
Spring передаст это имя в `ViewResolver`, который найдет файл `index.html` (или JSP `index.jsp`, или Thymeleaf-шаблон) и отрендерит страницу.

## 2. `@RestController`

Это **сокращение**: `@Controller + @ResponseBody`. Методы такого контроллера **по умолчанию возвращают данные**, а не имена представлений. Обычно используется для REST API (JSON/XML). То есть разница только в том, что `@RestController` автоматически навешивает `@ResponseBody` на все методы класса.
