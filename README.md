This project aims to facilitate the execution of native queries in the database.

The idea of ​​the project is to run queries by convention, similar to Spring Data.

When creating a new interface that extends the NativeQuery interface, we create false objects of these interfaces, which we use proxy to intercept the method calls and execute the queries, at the end we register the beans of these interfaces dynamically, thus being able to inject the interfaces in all components of the Spring.

The convention works the following way, the method name is the name of the file that will contain the query sql, the parameters of the methods will be passed as parameters to the entity manager, the return of the method is the object that will be transformed with the return of the Query.

The file that contains the SQL query is a Jtwig template, where we can apply validations in the query, a practical example to explain why we use template and not just a file containing SQL simply is the following, if you have a dynamic query, that conforms the values ​​of the methods you want to apply add or not a WHERE in the query, or modify it by completeness. The parameters of the methods besides being passed to the entity manager to execute the query are sent to the template.

Files with queries must be added to a folder named "nativeQuery" inside the resources folder. Remember, the file name must be the same as the method name.

Here is an example of using the framework.

> UserTO file example
```java
@Data
public class UserTO {
  private Number id;
  private String name;
}
```

> UserNativeQUery file example
```java
import br.com.viasoft.NativeQuery;
import java.util.List;
import org.springframework.data.domain.Pageable;

public interface UserNativeQuery implementes NativeQuery {

  List<UserTO> findUsers();
  
  List<UserTO> findActiveUsers(Pageagle pageable);
  
  UserTO findUserById(Number id);
  
}
```

> findUsers.twig file example
```sql
SELECT id, name FROM USER
```

> findActiveUsers.twig file example
```sql
SELECT id, name FROM USER WHERE ACTIVE = true
```

> findUserById.twig file example
```sql
SELECT id, name FROM USER WHERE id = :id
```

> UserController file example
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.PageRequest;
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("user")
public class UserController {

  @Autowired private UserNativeQuery userNativeQuery;
  
  @GetMapping()
  public List<UserTO> findUsers() {
    return userNativeQuery.findUsers();
  }
  
  @GetMapping("active")
  public List<UserTO> findUsers(@RequestParam(value = "page", defaultValue = "0") int page, @RequestParam(value = "size", defaultValue = "0") int size) {
    return userNativeQuery.findActiveUsers(PageRequest.of(page, size));
  }
  
  @GetMapping("{id}")
  public UserTO findUsers(@PathVariable("id") Number id) {
    return userNativeQuery.findUserById(id);
  }

}
```