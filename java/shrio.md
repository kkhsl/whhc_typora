# 会话管理

Apache Shiro 的登录时间管理是通过会话（Session）和会话管理器（Session Manager）来实现的。每次用户成功登录时，Shiro 会为该用户创建一个会话，并将登录时间、会话超时等信息保存在会话中。下面是 Shiro 登录时间管理的几个关键方面：

### 1. **会话管理（Session Management）**

Shiro 默认会为每个用户创建一个会话，登录时间会保存在会话中。Shiro 提供了多种会话管理方式，包括基于内存的会话管理和分布式会话管理。

- **会话对象**：`org.apache.shiro.session.Session` 是 Shiro 的会话接口，包含会话创建时间、最后一次访问时间、会话超时时间等。
- **会话管理器**：Shiro 通过 `SessionManager` 管理会话生命周期。默认情况下，Shiro 使用 `DefaultWebSessionManager` 来管理 Web 应用中的会话。

### 2. **获取登录时间**

在用户成功登录后，可以通过 `Session` 对象获取登录时间（即会话的创建时间）。

```
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.session.Session;
import org.apache.shiro.subject.Subject;

public class LoginTimeUtil {

    public static long getLoginTime() {
        Subject currentUser = SecurityUtils.getSubject();
        Session session = currentUser.getSession();
        return session.getStartTimestamp().getTime();  // 返回会话创建时间（登录时间）
    }
}
```

`getStartTimestamp()` 方法返回用户会话的开始时间，这通常是用户成功登录的时间。

### 3. **会话超时时间**

Shiro 会话超时是指用户在指定时间内未进行任何操作后，会话会自动失效。默认的会话超时时间为 30 分钟（1800000 毫秒）。你可以根据需求自定义会话超时时间。

#### 配置会话超时时间的方式：

- **方式 1：配置文件（如 `application.yml` 或 `shiro.ini`）**

  在 Spring Boot 中，你可以通过配置文件来设置 Shiro 会话超时时间：

  ```
  shiro:
    session:
      globalSessionTimeout: 3600000  # 会话超时时间，单位为毫秒，这里设置为 1 小时
  ```

- **方式 2：Java 代码配置**

  你也可以通过 Java 代码来配置会话超时时间：

  ```
  java复制代码@Bean
  public DefaultWebSessionManager sessionManager() {
      DefaultWebSessionManager sessionManager = new DefaultWebSessionManager();
      sessionManager.setGlobalSessionTimeout(3600000);  // 设置会话超时时间为1小时
      return sessionManager;
  }
  ```

### 4. **会话超时后的处理**

当会话超时后，Shiro 会自动使该用户的会话失效，用户需要重新登录。你可以通过 `SessionListener` 监听会话的创建、失效等事件，并根据业务需求做相应处理。

#### 示例：自定义会话监听器

```
import org.apache.shiro.session.Session;
import org.apache.shiro.session.SessionListener;

public class CustomSessionListener implements SessionListener {

    @Override
    public void onStart(Session session) {
        // 会话创建时触发
        System.out.println("Session started: " + session.getId());
    }

    @Override
    public void onStop(Session session) {
        // 会话停止时触发
        System.out.println("Session stopped: " + session.getId());
    }

    @Override
    public void onExpiration(Session session) {
        // 会话过期时触发
        System.out.println("Session expired: " + session.getId());
    }
}
```

你可以将 `CustomSessionListener` 注入到 `SessionManager` 中来监听会话事件：

```
@Bean
public DefaultWebSessionManager sessionManager() {
    DefaultWebSessionManager sessionManager = new DefaultWebSessionManager();
    Collection<SessionListener> listeners = new ArrayList<>();
    listeners.add(new CustomSessionListener());  // 添加自定义会话监听器
    sessionManager.setSessionListeners(listeners);
    return sessionManager;
}
```

### 5. **“记住我”功能与登录时间**

Shiro 还支持“记住我”功能，它允许用户在下次访问时无需重新登录。此功能通过 `RememberMeManager` 来管理。与会话不同，“记住我”功能并不会为用户创建新的会话，而是通过客户端的 Cookie 来实现。

- 如果用户通过“记住我”登录，Shiro 不会记录新的登录时间。
- 通过 `subject.isRemembered()` 可以判断用户是否通过“记住我”功能登录。

```
Subject currentUser = SecurityUtils.getSubject();
if (currentUser.isRemembered()) {
    System.out.println("User logged in via 'Remember Me'");
}
```

### 6. **SessionManager 的类型**

Shiro 提供了不同类型的 `SessionManager` 来处理不同场景下的会话管理需求：

- **`DefaultWebSessionManager`**：用于管理 Web 应用中的会话，支持会话的创建、销毁、超时、持久化等功能。
- **`EnterpriseCacheSessionDAO`**：用于缓存会话数据，适合分布式应用。
- **`ServletContainerSessionManager`**：使用 Servlet 容器的会话管理机制，而不是 Shiro 的会话机制，适合使用 Servlet 原生会话的项目。

### 总结

- **登录时间**：Shiro 登录时间通过 `Session` 的创建时间来管理，可以通过 `session.getStartTimestamp()` 获取登录时间。
- **会话超时**：默认超时时间为 30 分钟，可以通过配置文件或代码调整。
- **会话监听器**：可以自定义监听器来处理会话的开始、停止和超时事件。
- **“记住我”功能**：不会创建新的会话，因此不涉及登录时间的更新。

通过灵活配置会话管理，Shiro 能够有效地管理用户登录时间和会话生命周期

# 会话

在 Apache Shiro 中，会话过期后，默认情况下会话不会被自动续期。要实现会话的续期（即延长会话的有效期），你需要通过自定义代码或配置来处理。以下是一些常见的方式来实现会话续期：

### 1. **配置会话超时时间**

虽然这不是“续期”的直接方法，但你可以通过设置较长的会话超时时间来减少会话过期的可能性。

```
java复制代码@Bean
public DefaultWebSessionManager sessionManager() {
    DefaultWebSessionManager sessionManager = new DefaultWebSessionManager();
    sessionManager.setGlobalSessionTimeout(3600000); // 设置会话超时时间为1小时
    return sessionManager;
}
```

### 2. **自动续期**

自动续期通常涉及到会话活动时延长会话的有效期。你可以通过监听会话活动事件来实现。

#### 自定义会话监听器

你可以实现 `SessionListener` 接口，在会话访问时更新会话的最后访问时间，从而实现续期。

```
java复制代码import org.apache.shiro.session.Session;
import org.apache.shiro.session.SessionListener;
import org.apache.shiro.session.SessionContext;

public class AutoRenewSessionListener implements SessionListener {

    @Override
    public void onStart(Session session) {
        // 会话创建时
    }

    @Override
    public void onStop(Session session) {
        // 会话停止时
    }

    @Override
    public void onExpiration(Session session) {
        // 会话过期时
    }

    // 自定义方法来更新会话的最后访问时间
    public void renewSession(Session session) {
        session.setLastAccessTime(new Date());
    }
}
```

#### 在过滤器中续期

在自定义过滤器中，你可以手动更新会话的最后访问时间。

```
java复制代码import org.apache.shiro.web.filter.AccessControlFilter;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import org.apache.shiro.session.Session;
import org.apache.shiro.SecurityUtils;
import java.io.IOException;

public class SessionRenewalFilter extends AccessControlFilter {

    @Override
    protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) throws Exception {
        // 访问资源前续期会话
        renewSession();
        return true;
    }

    private void renewSession() {
        Session session = SecurityUtils.getSubject().getSession();
        session.setLastAccessTime(new Date());
    }

    @Override
    protected boolean onAccessDenied(ServletRequest request, ServletResponse response) throws Exception {
        // 未认证或未授权的处理逻辑
        return false;
    }
}
```

### 3. **用户活动续期**

可以在每次用户活动时（例如请求处理）检查会话的有效性，并根据需要更新会话的超时时间。这通常在自定义过滤器或会话管理器中实现。

### 4. **通过 Cookie 续期**

在一些应用中，可以通过更新客户端的 Cookie 来实现续期。虽然 Shiro 自身不直接提供这种方式，但你可以在应用逻辑中设置或更新 Cookie，以支持会话续期。

```
java复制代码import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServletResponse;

public class CookieUtils {

    public static void updateSessionCookie(HttpServletResponse response) {
        Cookie sessionCookie = new Cookie("SESSION_ID", getNewSessionId());
        sessionCookie.setMaxAge(3600); // 1小时
        response.addCookie(sessionCookie);
    }

    private static String getNewSessionId() {
        // 生成新的会话 ID 或从会话管理器中获取新的会话 ID
        return "newSessionId";
    }
}
```

### 5. **使用 Redis 或其他分布式缓存**

如果你使用 Redis 或其他分布式缓存存储会话信息，你可以通过配置缓存的过期时间来实现续期。当会话信息被存储在缓存中时，你可以通过设置更长的过期时间来间接实现续期。

### 6. **示例：配置会话续期**

如果你使用 `DefaultWebSessionManager`，可以在会话活动时更新会话的超时时间：

```
java复制代码import org.apache.shiro.session.mgt.DefaultWebSessionManager;
import org.apache.shiro.session.mgt.SessionContext;

@Bean
public DefaultWebSessionManager sessionManager() {
    DefaultWebSessionManager sessionManager = new DefaultWebSessionManager();
    sessionManager.setGlobalSessionTimeout(3600000); // 设置会话超时时间
    sessionManager.setSessionIdUrlRewritingEnabled(false); // 禁用会话 ID URL 重写
    sessionManager.setSessionValidationSchedulerEnabled(true); // 启用会话验证调度
    return sessionManager;
}
```

### 总结

- **配置超时时间**：直接设置较长的超时时间。
- **自动续期**：在会话访问时手动更新会话的最后访问时间。
- **用户活动续期**：在用户每次活动时检查并更新会话。
- **Cookie 续期**：通过更新客户端的 Cookie 来间接实现会话续期。
- **分布式缓存**：使用 Redis 或其他缓存来管理会话并设置过期时间。

根据应用的需求和架构选择合适的会话续期方法来确保用户体验。

# 过滤器

### 1. **认证（Authentication）过滤器**

用于确保用户已经通过认证才能访问资源。

- **`anon`**（Anonymous Filter）：允许匿名访问，不需要认证或登录。
- **`authc`**（Form Authentication Filter）：要求用户必须登录才能访问资源，未登录时会跳转到登录页面。
- **`authcBasic`**（HTTP Basic Authentication Filter）：使用 HTTP Basic 认证，适用于 REST 风格的 API。
- **`user`**（User Filter）：如果用户通过“记住我”功能登录，允许访问；否则要求用户登录。

### 2. **授权（Authorization）过滤器**

用于控制用户对资源的访问权限，基于角色和权限的访问控制。

- `roles`

  ：要求用户必须具备指定的角色才能访问资源。

  - 例如：`filterChainDefinitionMap.put("/admin/**", "roles[admin]");`

- `perms`

  ：要求用户具备指定的权限才能访问资源。

  - 例如：`filterChainDefinitionMap.put("/documents/**", "perms[document:read]");`

### 3. **会话管理（Session Management）过滤器**

用于处理会话相关的功能，如退出登录等。

- `logout`

  ：处理用户注销登录的过滤器，清除用户的身份信息，并重定向到登录页面或指定的 URL。

  - 例如：`filterChainDefinitionMap.put("/logout", "logout");`

### 4. **JSecurity 过滤器**

这些过滤器继承了 Java EE 安全规范中的经典过滤器行为。

- **`ssl`**：要求通过 SSL（HTTPS）访问资源。如果请求不是 HTTPS，将重定向到 HTTPS。

- `port`

  ：要求请求使用指定的端口号访问。如果请求端口不符合要求，会重定向到正确的端口。

  - 例如：`filterChainDefinitionMap.put("/secure/**", "port[8443]");`

### 5. **常用 Shiro 内置过滤器总结**

| 过滤器       | 功能描述                                             |
| ------------ | ---------------------------------------------------- |
| `anon`       | 允许匿名访问，不需要认证。                           |
| `authc`      | 要求用户必须登录才能访问，未登录会跳转到登录页面。   |
| `authcBasic` | 基于 HTTP Basic 认证进行访问，通常用于 REST API。    |
| `user`       | 用户可以通过“记住我”功能登录，允许访问。             |
| `logout`     | 处理用户注销登录，清除身份信息并重定向到指定页面。   |
| `roles`      | 要求用户拥有指定的角色才能访问资源。                 |
| `perms`      | 要求用户拥有指定的权限才能访问资源。                 |
| `ssl`        | 要求使用 HTTPS 访问资源。                            |
| `port`       | 要求使用指定的端口访问资源，自动重定向到正确的端口。 |

### 6. **自定义过滤器**

除了内置的过滤器，Shiro 允许你自定义过滤器。你可以继承 `AccessControlFilter` 或 `AuthenticatingFilter`，并实现自定义的认证或授权逻辑。

例如，一个自定义的权限控制过滤器：

```
java复制代码public class CustomPermissionFilter extends AccessControlFilter {
    
    @Override
    protected boolean isAccessAllowed(ServletRequest request, ServletResponse response, Object mappedValue) throws Exception {
        // 自定义权限控制逻辑
        Subject subject = getSubject(request, response);
        String[] perms = (String[]) mappedValue;
        return perms != null && perms.length > 0 && subject.isPermittedAll(perms);
    }

    @Override
    protected boolean onAccessDenied(ServletRequest request, ServletResponse response) throws Exception {
        // 处理未授权逻辑
        response.getWriter().write("You do not have permission to access this resource.");
        return false;
    }
}
```

### 7. **过滤器链的使用**

你可以通过 `ShiroFilterFactoryBean` 设置 Shiro 的过滤器链，来配置不同 URL 使用的过滤器。例如：

```
java复制代码Map<String, String> filterChainDefinitionMap = new LinkedHashMap<>();
filterChainDefinitionMap.put("/login", "anon");  // 登录页面允许匿名访问
filterChainDefinitionMap.put("/admin/**", "authc, roles[admin]");  // /admin 需要登录并且具有 admin 角色
filterChainDefinitionMap.put("/documents/**", "authc, perms[document:read]");  // 需要 document:read 权限
filterChainDefinitionMap.put("/logout", "logout");  // 注销过滤器
shiroFilterFactoryBean.setFilterChainDefinitionMap(filterChainDefinitionMap);
```

### 总结

Shiro 提供了多种内置过滤器，涵盖认证、授权、会话管理等功能。你可以根据应用的具体需求，通过过滤器链配置来灵活控制用户的访问权限，同时还可以自定义过滤器来满足特定的业务需求。