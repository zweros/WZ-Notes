---
title: 20190813 Spring Security
---

# Spring Security #

>  Spring Security is a powerful and highly customizable authentication and access-control framework. 
>
> It is the de-facto standard for securing Spring-based applications.
>
> Spring Security is a framework that focuses on providing both authentication and authorization to Java applications. Like all Spring projects, the real power of Spring Security is found in how easily it can be extended to meet custom requirements

Spring Security 前身是 The Acegi Security System for  Spring 项目

## Spring Security  入门项目 ##

- pom 文件

```xml
<!-- Spring Security -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

- 配置类

```java
/**
 *  Spring Security 配置
 */
@Configuration
public class SecurityConfig {

    /**
     * 将创建的 BCryptPasswordEncoder 对象放入 Spring IOC 容器中
     * @return
     */
    @Bean
    public PasswordEncoder getPasswordEncoder(){
        return new BCryptPasswordEncoder();
    }
}
```

## Spring Security 相关 API  ##

### UserDetailsService 接口 ###

```java
public interface UserDetailsService {

	/**
    *Locates the user based on the username. In the actual implementation, the search
	* may possibly be case sensitive, or case insensitive depending on how the
	* implementation instance is configured.
	**/
	UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}

```

### User 类 ###

```java
/*
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
public class User implements UserDetails, CredentialsContainer {

	private static final long serialVersionUID = SpringSecurityCoreVersion.SERIAL_VERSION_UID;

	private static final Log logger = LogFactory.getLog(User.class);

	private String password;
	private final String username;
	private final Set<GrantedAuthority> authorities;
	private final boolean accountNonExpired;
	private final boolean accountNonLocked;
	private final boolean credentialsNonExpired;
    
	/**
	 * Calls the more complex constructor with all boolean arguments set to {@code true}.
	 */
	public User(String username, String password,
			Collection<? extends GrantedAuthority> authorities) {
		this(username, password, true, true, true, true, authorities);
	}

	/**
	 * Construct the <code>User</code> with the details required by
	 * {@link org.springframework.security.authentication.dao.DaoAuthenticationProvider}.
	 *
	 * @param username the username presented to the
	 * <code>DaoAuthenticationProvider</code>
	 * @param password the password that should be presented to the
	 * <code>DaoAuthenticationProvider</code>
	 * @param enabled set to <code>true</code> if the user is enabled
	 * @param accountNonExpired set to <code>true</code> if the account has not expired
	 * @param credentialsNonExpired set to <code>true</code> if the credentials have not
	 * expired
	 * @param accountNonLocked set to <code>true</code> if the account is not locked
	 * @param authorities the authorities that should be granted to the caller if they
	 * presented the correct username and password and the user is enabled. Not null.
	 *
	 * @throws IllegalArgumentException if a <code>null</code> value was passed either as
	 * a parameter or as an element in the <code>GrantedAuthority</code> collection
	 */
	public User(String username, String password, boolean enabled,
			boolean accountNonExpired, boolean credentialsNonExpired,
			boolean accountNonLocked, Collection<? extends GrantedAuthority> authorities) {

		if (((username == null) || "".equals(username)) || (password == null)) {
			throw new IllegalArgumentException(
					"Cannot pass null or empty values to constructor");
		}

		this.username = username;
		this.password = password;
		this.enabled = enabled;
		this.accountNonExpired = accountNonExpired;
		this.credentialsNonExpired = credentialsNonExpired;
		this.accountNonLocked = accountNonLocked;
		this.authorities = Collections.unmodifiableSet(sortAuthorities(authorities));
	}


	public Collection<GrantedAuthority> getAuthorities() {
		return authorities;
	}

	public String getPassword() {
		return password;
	}

	public String getUsername() {
		return username;
	}

	public boolean isEnabled() {
		return enabled;
	}

	public boolean isAccountNonExpired() {
		return accountNonExpired;
	}

	public boolean isAccountNonLocked() {
		return accountNonLocked;
	}

	public boolean isCredentialsNonExpired() {
		return credentialsNonExpired;
	}

	public void eraseCredentials() {
		password = null;
	}

	private static SortedSet<GrantedAuthority> sortAuthorities(
			Collection<? extends GrantedAuthority> authorities) {
		Assert.notNull(authorities, "Cannot pass a null GrantedAuthority collection");
		// Ensure array iteration order is predictable (as per
		// UserDetails.getAuthorities() contract and SEC-717)
		SortedSet<GrantedAuthority> sortedAuthorities = new TreeSet<>(
				new AuthorityComparator());

		for (GrantedAuthority grantedAuthority : authorities) {
			Assert.notNull(grantedAuthority,
					"GrantedAuthority list cannot contain any null elements");
			sortedAuthorities.add(grantedAuthority);
		}

		return sortedAuthorities;
	}

	private static class AuthorityComparator implements Comparator<GrantedAuthority>,
			Serializable {
		private static final long serialVersionUID = SpringSecurityCoreVersion.SERIAL_VERSION_UID;

		public int compare(GrantedAuthority g1, GrantedAuthority g2) {
			// Neither should ever be null as each entry is checked before adding it to
			// the set.
			// If the authority is null, it is a custom authority and should precede
			// others.
			if (g2.getAuthority() == null) {
				return -1;
			}

			if (g1.getAuthority() == null) {
				return 1;
			}

			return g1.getAuthority().compareTo(g2.getAuthority());
		}
	}

	/**
	 * Returns {@code true} if the supplied object is a {@code User} instance with the
	 * same {@code username} value.
	 * <p>
	 * In other words, the objects are equal if they have the same username, representing
	 * the same principal.
	 */
	@Override
	public boolean equals(Object rhs) {
		if (rhs instanceof User) {
			return username.equals(((User) rhs).username);
		}
		return false;
	}

	/**
	 * Returns the hashcode of the {@code username}.
	 */
	@Override
	public int hashCode() {
		return username.hashCode();
	}

	@Override
	public String toString() {
		StringBuilder sb = new StringBuilder();
		sb.append(super.toString()).append(": ");
		sb.append("Username: ").append(this.username).append("; ");
		sb.append("Password: [PROTECTED]; ");
		sb.append("Enabled: ").append(this.enabled).append("; ");
		sb.append("AccountNonExpired: ").append(this.accountNonExpired).append("; ");
		sb.append("credentialsNonExpired: ").append(this.credentialsNonExpired)
				.append("; ");
		sb.append("AccountNonLocked: ").append(this.accountNonLocked).append("; ");

		if (!authorities.isEmpty()) {
			sb.append("Granted Authorities: ");

			boolean first = true;
			for (GrantedAuthority auth : authorities) {
				if (!first) {
					sb.append(",");
				}
				first = false;

				sb.append(auth);
			}
		}
		else {
			sb.append("Not granted any authorities");
		}

		return sb.toString();
	}

	/**
	 * Creates a UserBuilder with a specified user name
	 *
	 * @param username the username to use
	 * @return the UserBuilder
	 */
	public static UserBuilder withUsername(String username) {
		return builder().username(username);
	}

	/**
	 * Creates a UserBuilder
	 *
	 * @return the UserBuilder
	 */
	public static UserBuilder builder() {
		return new UserBuilder();
	}

	/**
	 * <p>
	 * <b>WARNING:</b> This method is considered unsafe for production and is only intended
	 * for sample applications.
	 * </p>
	 * <p>
	 * Creates a user and automatically encodes the provided password using
	 * {@code PasswordEncoderFactories.createDelegatingPasswordEncoder()}. For example:
	 * </p>
	 *
	 * <pre>
	 * <code>
	 * UserDetails user = User.withDefaultPasswordEncoder()
	 *     .username("user")
	 *     .password("password")
	 *     .roles("USER")
	 *     .build();
	 * // outputs {bcrypt}$2a$10$dXJ3SW6G7P50lGmMkkmwe.20cQQubK3.HZWzG3YB1tlRy.fqvM/BG
	 * System.out.println(user.getPassword());
	 * </code>
	 * </pre>
	 *
	 * This is not safe for production (it is intended for getting started experience)
	 * because the password "password" is compiled into the source code and then is
	 * included in memory at the time of creation. This means there are still ways to
	 * recover the plain text password making it unsafe. It does provide a slight
	 * improvement to using plain text passwords since the UserDetails password is
	 * securely hashed. This means if the UserDetails password is accidentally exposed,
	 * the password is securely stored.
	 *
	 * In a production setting, it is recommended to hash the password ahead of time.
	 * For example:
	 *
	 * <pre>
	 * <code>
	 * PasswordEncoder encoder = PasswordEncoderFactories.createDelegatingPasswordEncoder();
	 * // outputs {bcrypt}$2a$10$dXJ3SW6G7P50lGmMkkmwe.20cQQubK3.HZWzG3YB1tlRy.fqvM/BG
	 * // remember the password that is printed out and use in the next step
	 * System.out.println(encoder.encode("password"));
	 * </code>
	 * </pre>
	 *
	 * <pre>
	 * <code>
	 * UserDetails user = User.withUsername("user")
	 *     .password("{bcrypt}$2a$10$dXJ3SW6G7P50lGmMkkmwe.20cQQubK3.HZWzG3YB1tlRy.fqvM/BG")
	 *     .roles("USER")
	 *     .build();
	 * </code>
	 * </pre>
	 *
	 * @return a UserBuilder that automatically encodes the password with the default
	 * PasswordEncoder
	 * @deprecated Using this method is not considered safe for production, but is
	 * acceptable for demos and getting started. For production purposes, ensure the
	 * password is encoded externally. See the method Javadoc for additional details.
	 * There are no plans to remove this support. It is deprecated to indicate
	 * that this is considered insecure for production purposes.
	 */
	@Deprecated
	public static UserBuilder withDefaultPasswordEncoder() {
		logger.warn("User.withDefaultPasswordEncoder() is considered unsafe for production and is only intended for sample applications.");
		PasswordEncoder encoder = PasswordEncoderFactories.createDelegatingPasswordEncoder();
		return builder().passwordEncoder(encoder::encode);
	}

	public static UserBuilder withUserDetails(UserDetails userDetails) {
		return withUsername(userDetails.getUsername())
			.password(userDetails.getPassword())
			.accountExpired(!userDetails.isAccountNonExpired())
			.accountLocked(!userDetails.isAccountNonLocked())
			.authorities(userDetails.getAuthorities())
			.credentialsExpired(!userDetails.isCredentialsNonExpired())
			.disabled(!userDetails.isEnabled());
	}

	/**
	 * Builds the user to be added. At minimum the username, password, and authorities
	 * should provided. The remaining attributes have reasonable defaults.
	 */
	public static class UserBuilder {
		private String username;
		private String password;
		private List<GrantedAuthority> authorities;
		private boolean accountExpired;
		private boolean accountLocked;
		private boolean credentialsExpired;
		private boolean disabled;
		private Function<String, String> passwordEncoder = password -> password;

		/**
		 * Creates a new instance
		 */
		private UserBuilder() {
		}

		/**
		 * Populates the username. This attribute is required.
		 *
		 * @param username the username. Cannot be null.
		 * @return the {@link UserBuilder} for method chaining (i.e. to populate
		 * additional attributes for this user)
		 */
		public UserBuilder username(String username) {
			Assert.notNull(username, "username cannot be null");
			this.username = username;
			return this;
		}

		/**
		 * Populates the password. This attribute is required.
		 *
		 * @param password the password. Cannot be null.
		 * @return the {@link UserBuilder} for method chaining (i.e. to populate
		 * additional attributes for this user)
		 */
		public UserBuilder password(String password) {
			Assert.notNull(password, "password cannot be null");
			this.password = password;
			return this;
		}

	
		public UserBuilder passwordEncoder(Function<String, String> encoder) {
			Assert.notNull(encoder, "encoder cannot be null");
			this.passwordEncoder = encoder;
			return this;
		}

		
		public UserBuilder roles(String... roles) {
			List<GrantedAuthority> authorities = new ArrayList<>(
					roles.length);
			for (String role : roles) {
				Assert.isTrue(!role.startsWith("ROLE_"), () -> role
						+ " cannot start with ROLE_ (it is automatically added)");
				authorities.add(new SimpleGrantedAuthority("ROLE_" + role));
			}
			return authorities(authorities);
		}

	
		public UserBuilder authorities(GrantedAuthority... authorities) {
			return authorities(Arrays.asList(authorities));
		}

	
		public UserBuilder authorities(Collection<? extends GrantedAuthority> authorities) {
			this.authorities = new ArrayList<>(authorities);
			return this;
		}

	
		public UserBuilder authorities(String... authorities) {
			return authorities(AuthorityUtils.createAuthorityList(authorities));
		}

	
		public UserBuilder accountExpired(boolean accountExpired) {
			this.accountExpired = accountExpired;
			return this;
		}

		public UserBuilder accountLocked(boolean accountLocked) {
			this.accountLocked = accountLocked;
			return this;
		}


		public UserBuilder credentialsExpired(boolean credentialsExpired) {
			this.credentialsExpired = credentialsExpired;
			return this;
		}


	
		public UserBuilder disabled(boolean disabled) {
			this.disabled = disabled;
			return this;
		}

		public UserDetails build() {
			String encodedPassword = this.passwordEncoder.apply(password);
			return new User(username, encodedPassword, !disabled, !accountExpired,
					!credentialsExpired, !accountLocked, authorities);
		}
	}
}

```

### PasswordEncoder 接口 ###

> 该接口用于加密密码，推荐使用 BCryptPasswordEncoder 实现类

```java
**
 * Service interface for encoding passwords.
 *
 * The preferred implementation is {@code BCryptPasswordEncoder}.
 *
 * @author Keith Donald
 */
public interface PasswordEncoder {

	/**
	 * Encode the raw password. Generally, a good encoding algorithm applies a SHA-1 or
	 * greater hash combined with an 8-byte or greater randomly generated salt.
	 */
	String encode(CharSequence rawPassword);

	/**
	 * Verify the encoded password obtained from storage matches the submitted raw
	 * password after it too is encoded. Returns true if the passwords match, false if
	 * they do not. The stored password itself is never decoded.
	 *
	 * @param rawPassword the raw password to encode and match
	 * @param encodedPassword the encoded password from storage to compare with
	 * @return true if the raw password, after encoding, matches the encoded password from
	 * storage
	 */
	boolean matches(CharSequence rawPassword, String encodedPassword);

	/**
	 * Returns true if the encoded password should be encoded again for better security,
	 * else false. The default implementation always returns false.
	 * @param encodedPassword the encoded password to check
	 * @return true if the encoded password should be encoded again for better security,
	 * else false.
	 */
	default boolean upgradeEncoding(String encodedPassword) {
		return false;
	}
}

```

### BCryptPasswordEncoder 类-密码解析器 ###

```java
/**
 * Implementation of PasswordEncoder that uses the BCrypt strong hashing function. Clients
 * can optionally supply a "strength" (a.k.a. log rounds in BCrypt) and a SecureRandom
 * instance. The larger the strength parameter the more work will have to be done
 * (exponentially) to hash the passwords. The default value is 10.
 *
 * @author Dave Syer
 *
 */
public class BCryptPasswordEncoder implements PasswordEncoder {
	private Pattern BCRYPT_PATTERN = Pattern
			.compile("\\A\\$2a?\\$\\d\\d\\$[./0-9A-Za-z]{53}");
	private final Log logger = LogFactory.getLog(getClass());

	private final int strength;

	private final SecureRandom random;

	public BCryptPasswordEncoder() {
		this(-1);
	}

	/**
	 * @param strength the log rounds to use, between 4 and 31
	 */
	public BCryptPasswordEncoder(int strength) {
		this(strength, null);
	}

	/**
	 * @param strength the log rounds to use, between 4 and 31
	 * @param random the secure random instance to use
	 *
	 */
	public BCryptPasswordEncoder(int strength, SecureRandom random) {
		if (strength != -1 && (strength < BCrypt.MIN_LOG_ROUNDS || strength > BCrypt.MAX_LOG_ROUNDS)) {
			throw new IllegalArgumentException("Bad strength");
		}
		this.strength = strength;
		this.random = random;
	}

	public String encode(CharSequence rawPassword) {
		String salt;
		if (strength > 0) {
			if (random != null) {
				salt = BCrypt.gensalt(strength, random);
			}
			else {
				salt = BCrypt.gensalt(strength);
			}
		}
		else {
			salt = BCrypt.gensalt();
		}
		return BCrypt.hashpw(rawPassword.toString(), salt);
	}

	public boolean matches(CharSequence rawPassword, String encodedPassword) {
		if (encodedPassword == null || encodedPassword.length() == 0) {
			logger.warn("Empty encoded password");
			return false;
		}

		if (!BCRYPT_PATTERN.matcher(encodedPassword).matches()) {
			logger.warn("Encoded password does not look like BCrypt");
			return false;
		}

		return BCrypt.checkpw(rawPassword.toString(), encodedPassword);
	}
}

```

## 认证模块 Authentication ##

### 自定义登录逻辑 ###
- UserDetailsServiceImpl.java

```java
@Service
public class UserDetailsServiceImpl implements UserDetailsService {

    @Autowired
    private PasswordEncoder encoder;

    @Override
    public UserDetails loadUserByUsername(String username) 
        throws UsernameNotFoundException {
        //判断用户名是否存在
        if(username ==null || !username.equals("admin")){
            throw new UsernameNotFoundException("该用户名不存在");
        }
        // 模拟从数据库中获取加密后的密码
        String password = encoder.encode("123");

        return 
            new User(username,
                     password, 
                     AuthorityUtils.
                     commaSeparatedStringToAuthorityList("admin,normal"));
    }
}

```

### 自定义登录界面 ###

- 修改配置类，继承 WebSecurityConfigurerAdapter 父类，并重写 configure 方法

```java
/**
 *  Spring Security 配置
 */
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    // 重写 configure 方法
    // 注意 configure 方法有两个，选择参数 HttpSecurity
    @Override
    protected void configure(HttpSecurity http) throws Exception {

           //表单控制
            http.formLogin()
                    .loginPage("/login.html")      // 登录界面
                    .loginProcessingUrl("/login") //拦截的登录请求，form 的 action 取值
                    .successForwardUrl("/toMain"); // post 请求，跳转到登录成功界面
            // URL 认证
            http.authorizeRequests()
                    .antMatchers("/login.html").permitAll()  // login.html 被执行
                    .anyRequest().authenticated();           // 所有请求都需被认证
        
            //关闭 csrf 防护
            http.csrf().disable();
    }

    /**
     * 将创建的 BCryptPasswordEncoder 对象放入 Spring IOC 容器中
     * @return
     */
    @Bean
    public PasswordEncoder getPasswordEncoder(){
        return new BCryptPasswordEncoder();
    }
}

```

### 自定义失败界面 ###

- 修改 Spring Security 配置类

  添加登录失败的后跳转的 URL,以及放行登录失败界面

```java
/**
 *  Spring Security 配置
 */
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    // 重写 configure 方法
    // 注意 configure 方法有两个，选择参数 HttpSecurity
    @Override
    protected void configure(HttpSecurity http) throws Exception {

           //表单控制
            http.formLogin()
                    .loginPage("/login.html")         // 登录界面
                    .failureForwardUrl("/tofail")    //拦截登录失败请求... ，同样必须 post 请求
                    .loginProcessingUrl("/login")    //拦截的登录请求，form 的 action 取值
                    .successForwardUrl("/toMain");   // post 请求，跳转到登录成功界面
            // URL 认证
            http.authorizeRequests()
                    .antMatchers("/login.html").permitAll()  // 放行 login.html
                    .antMatchers("/fail.html").permitAll()   // 放行 fail.html
                    .anyRequest().authenticated();           // 所有请求都需被认证

            //关闭 csrf 防护
            http.csrf().disable();
    }

    /**
     * 将创建的 BCryptPasswordEncoder 对象放入 Spring IOC 容器中
     * @return
     */
    @Bean
    public PasswordEncoder getPasswordEncoder(){
        return new BCryptPasswordEncoder();
    }
}

```

- LoginControler 类

```java
@PostMapping("/tofail")
public String tofail(){
    return "redirect:/fail.html";
}
```

### 设置表单请求账号名和密码的参数名 ###

当进行登录时会执行 `UsernamePasswordAuthenticationFilter` 过滤器，其中要求表单中的用户名为 username ，密码为 password。这里可以通过配置类修改账号名和密码的参数名称

- UsernamePasswordAuthenticationFilter 过滤器类

```java
public class UsernamePasswordAuthenticationFilter extends
		AbstractAuthenticationProcessingFilter {

	public static final String SPRING_SECURITY_FORM_USERNAME_KEY = "username";
	public static final String SPRING_SECURITY_FORM_PASSWORD_KEY = "password";

	private String usernameParameter = SPRING_SECURITY_FORM_USERNAME_KEY;
	private String passwordParameter = SPRING_SECURITY_FORM_PASSWORD_KEY;
	private boolean postOnly = true;

	public UsernamePasswordAuthenticationFilter() {
		super(new AntPathRequestMatcher("/login", "POST"));
	}

	public Authentication attemptAuthentication(HttpServletRequest request,
			HttpServletResponse response) throws AuthenticationException {
		if (postOnly && !request.getMethod().equals("POST")) {
			throw new AuthenticationServiceException(
					"Authentication method not supported: " + request.getMethod());
		}

		String username = obtainUsername(request);
		String password = obtainPassword(request);

		if (username == null) {
			username = "";
		}

		if (password == null) {
			password = "";
		}

		username = username.trim();

		UsernamePasswordAuthenticationToken authRequest = new UsernamePasswordAuthenticationToken(
				username, password);

		// Allow subclasses to set the "details" property
		setDetails(request, authRequest);

		return this.getAuthenticationManager().authenticate(authRequest);
	}


	protected String obtainPassword(HttpServletRequest request) {
		return request.getParameter(passwordParameter);
	}


	protected String obtainUsername(HttpServletRequest request) {
		return request.getParameter(usernameParameter);
	}


}

```

### 修改配置类 ###

```java
/**
 *  Spring Security 配置
 */
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    // 重写 configure 方法
    // 注意 configure 方法有两个，选择参数 HttpSecurity
    @Override
    protected void configure(HttpSecurity http) throws Exception {

           //表单控制
            http.formLogin()
                    .usernameParameter("name")        //自定义表单参数用户名
                    .passwordParameter("pwd")         //自定义表单参数密码
                    .loginPage("/login.html")         // 登录界面
                    .failureForwardUrl("/tofail")    //拦截登录失败请求... ，同样必须 post 请求
                    .loginProcessingUrl("/login")    //拦截的登录请求，form 的 action 取值
                    .successForwardUrl("/toMain");   // post 请求，跳转到登录成功界面
    }
}
```

### 自定义成功页面跳转 ###

> 源码中使用站内跳转，而不能实现站外跳转
>
> 自定义成功页面跳转，必须使用 POST 请求

- 源码分析

```java
// FormLoginConfigurer 类
public FormLoginConfigurer<H> successForwardUrl(String forwardUrl) {
        this.successHandler(new ForwardAuthenticationSuccessHandler(forwardUrl));
        return this;
 }

// ForwardAuthenticationSuccessHandler 类
public class ForwardAuthenticationSuccessHandler implements AuthenticationSuccessHandler {

	private final String forwardUrl;


	public ForwardAuthenticationSuccessHandler(String forwardUrl) {
		Assert.isTrue(UrlUtils.isValidRedirectUrl(forwardUrl),
				() -> "'" + forwardUrl + "' is not a valid forward URL");
		this.forwardUrl = forwardUrl;
	}

	public void onAuthenticationSuccess(HttpServletRequest request, 
         		HttpServletResponse response, 
         	Authentication authentication) throws IOException, ServletException
    {
		request.getRequestDispatcher(forwardUrl).forward(request, response);
	}
}

```

- 自定义实现 AuthenticationSuccessHandler 接口

```java
public class MyAuthenticationSuccessHandler implements AuthenticationSuccessHandler {

    private  final String url;  //登录成功后，跳转到的 URL 地址

    public MyAuthenticationSuccessHandler(String url){
        this.url = url;
    }

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
        // 获取认证的主体 ，User
        User user = (User) authentication.getPrincipal();
        System.out.println(user.getUsername());
        System.out.println(user.getPassword()); //输出为 n ull
        System.out.println(user.getAuthorities()); //权限信息
        response.sendRedirect(url);
    }
}
```

- 修改配置类

```java
// 重写 configure 方法
    // 注意 configure 方法有两个，选择参数 HttpSecurity
    @Override
    protected void configure(HttpSecurity http) throws Exception {

           //表单控制
           http.formLogin()
         .usernameParameter("name")        //自定义表单参数用户名
         .passwordParameter("pwd")         //自定义表单参数密码
         .loginPage("/login.html")         // 登录界面
         .failureForwardUrl("/tofail")    //拦截登录失败请求... ，同样必须 post 请求
                .loginProcessingUrl("/login")    //拦截的登录请求，form 的 action 取值
                .successHandler(new MyAuthenticationSuccessHandler("http://www.baidu.com"));
    //    .successForwardUrl("/toMain");// post 请求，跳转到登录成功界面

        
    }

```

### 自定义失败页面跳转 ###

- 源码分析

```java
 public FormLoginConfigurer<H> failureForwardUrl(String forwardUrl) {
        this.failureHandler(new ForwardAuthenticationFailureHandler(forwardUrl));
        return this;
    }

public class ForwardAuthenticationFailureHandler implements AuthenticationFailureHandler {

	private final String forwardUrl;

	/**
	 * @param forwardUrl
	 */
	public ForwardAuthenticationFailureHandler(String forwardUrl) {
		Assert.isTrue(UrlUtils.isValidRedirectUrl(forwardUrl),
				() -> "'" + forwardUrl + "' is not a valid forward URL");
		this.forwardUrl = forwardUrl;
	}

	public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
		request.setAttribute(WebAttributes.AUTHENTICATION_EXCEPTION, exception);
		request.getRequestDispatcher(forwardUrl).forward(request, response);
	}
}
```

- 登录失败处理器

```java
**
 * 自定义登录失败处理器
 */
public class MyAuthenticationFailureHandler implements AuthenticationFailureHandler {

    private final String faiUrl;

    public MyAuthenticationFailureHandler(String faiUrl) {
        this.faiUrl = faiUrl;
    }

    @Override
    public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
        request.getSession().setAttribute("error",exception.getMessage());
        response.sendRedirect(this.faiUrl);
    }
}
```

- 修改配置类

```java
// 重写 configure 方法
// 注意 configure 方法有两个，选择参数 HttpSecurity
@Override
protected void configure(HttpSecurity http) throws Exception {

    //表单控制
    http.formLogin()
        .usernameParameter("name")        //自定义表单参数用户名
        .passwordParameter("pwd")         //自定义表单参数密码
        .loginPage("/login.html")         // 登录界面
        .failureHandler(new MyAuthenticationFailureHandler("/fail.html"))
        //     .failureForwardUrl("/tofail")    //拦截登录失败请求... ，同样必须 post 请求
        .loginProcessingUrl("/login")    //拦截的登录请求，form 的 action 取值
        .successHandler(new MyAuthenticationSuccessHandler("/main.html"));
    //    .successForwardUrl("/toMain");// post 请求，跳转到登录成功界面

 
}
```

##  授权模块 Authorization ##

### anyRequest()  ###

> 表示匹配所有请求

在所有匹配规则中去所有规则的交集

```java
.anyRequest().authenticated();          // 所有请求都需被认证，登录后才使用
```

### antMatchers(String ... antPatterns)  ###

> 参数是不定向参数，每个参数一个表示 ant 表达式，用于匹配 URL 规则

| antPatterns | 作用                      |
| ----------- | ------------------------- |
| ？          | 匹配任意1 个              |
| *           | 匹配任意0个或多个         |
| **          | 匹配0个目录及多个层级目录 |

- 修改配置类

```java
// 放行静态资源
.antMatchers("/css/**","/js/**","/imgs/**").permitAll()
.antMatchers("/imgs/*.jpg").permitAll()
```

### regexMatchers(String... regexPatterns)  ###

> 正则表达式匹配

```java
.regexMatchers(".+[.]jpg").permitAll()   //正则表达式匹配，所有以 jpg 结尾的文件都被放行 
//.表示一个 0个或多个 + 表示至少1个，[] 表示转译
```

### regexMatchers(HttpMethod method, String... regexPatterns)  ###

> 对 URL  绑定请求方式

```java
 .regexMatchers(HttpMethod.POST,"/demo").permitAll()
```

### mvcMatchers(String... patterns) ###

> 适用于配置了 servletPath 的情况
>
> serveltPath 就是所有的 URl 的统一前缀

在 springmvc 的项目中可以在 application.properties 添加如下内容设置 serveltPath 

```java
spring.mvc.servlet.path=/szxy
```

- 修改 配置类

```java
.mvcMatchers("/demo").servletPath("/szxy").permitAll()  //前缀 servletPath 前缀
.antMatchers("/szxy/demo").permitAll()  //等价于后面的配置
```

### 内置访问控制方法 ###

- ExpressionUrlAuthorizationConfigurer 源码分析

```java
static final String permitAll = "permitAll";
private static final String denyAll = "denyAll";
private static final String anonymous = "anonymous";
private static final String authenticated = "authenticated";
// fullyAuthenticated 表示通过用户登录界面进入的
private static final String fullyAuthenticated = "fullyAuthenticated";
// rememberMe 表示记住密码，用户直接跳转登录的
private static final String rememberMe = "rememberMe";
```

## 角色权限判断  ##

> 用于用户已经被认证成功后，判断用户是否具有特定的要求

### hasAuthority(String) ###

> 判断用户是否具有特定的权限，用户的权限是在自定义	登录逻辑中创建 User 对象时指定的

下图中 admin 就是用户的权限，admin 严格区分大小写

```java
  return
      new User(
      username,password, 
      AuthorityUtils.
      commaSeparatedStringToAuthorityList("admin,normal")
  );
```

- 修改配置类

```java
//.antMatchers("/main1.html").hasAuthority("admin")
 .antMatchers("/main1.html").hasAnyAuthority("admin1,admiN,admin")
```

### hasRole(String role) ###

> role 角色名严格区分大小写

- UserDetailsServiceImpl.java:

```java
// ROLE_ 开头表示权限
return new User(
    	username,
        password, 	
    	AuthorityUtils.
    	commaSeparatedStringToAuthorityList("admin,normal,ROLE_abc"));
```

- 修改配置类

```java
// .antMatchers("/main1.html").hasRole("abC")   // abc 区分大小写
.antMatchers("/main1.html").hasAnyRole("abC,abc")            
```

### hasIpAddress ###

> 指定 IP 访问

```java
System.out.println("getRemoteAddr："+request.getRemoteAddr());
//http://localhost:8080 登录
//getRemoteAddr：0:0:0:0:0:0:0:1
//htttp://127.0.0.1 
getRemoteAddr：127.0.0.1
//http://192.168.48.13:8080/main.html
//getRemoteAddr：192.168.48.13
```

- 修改配置类

```
 .antMatchers("/main1.html").hasIpAddress("127.0.0.1") //指定 iP 访问
```

### 自定义 403 页面 ###

- 自定义异常界面类

```java
/**
 * 加入到 Spring IOC 容器中
 */
@Component
public class MyAccessDeniedHandler implements AccessDeniedHandler {

    /**
     * 处理 403 异常界面
     * @param request
     * @param response
     * @param accessDeniedException
     * @throws IOException
     * @throws ServletException
     */
    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException, ServletException {
         // 设置响应状态码
        response.setStatus(HttpServletResponse.SC_FORBIDDEN); //403
        //  设置响应类型及编码
        response.setContentType("application/json;charset=utf-8");
        //  设置响应内容
        PrintWriter out = response.getWriter();
        out.print("{\"code\":\"403\",\"error\":\"访问受限,请与管理员联系\"}");
        String message = accessDeniedException.getMessage();
        System.out.println(message);
        out.flush();
        out.close();
    }
}
```

- 修改配置类

```java
 //异常处理
 http.
     exceptionHandling().
     accessDeniedHandler(new MyAccessDeniedHandler());
```

### Access  表达式 ###

> 之前的登录用户权限判断实际上底层实现都是调用 access 表达式

- 源码分析

```java
public 
ExpressionUrlAuthorizationConfigurer<H>.ExpressionInterceptUrlRegistry permitAll() 
{
            return this.access("permitAll");
}

 public ExpressionUrlAuthorizationConfigurer<H>.ExpressionInterceptUrlRegistry hasRole(String role) 
 {
       return this.
           access(ExpressionUrlAuthorizationConfigurer.hasRole(role));
}
```



- 修改配置类

```java
//  .antMatchers("/main1.html").access("permitAll")
.antMatchers("/main1.html").access("hasRole('abc')")
```



### 自定义业务逻辑接口 ###

- UserService.java

```java
public interface UserService {

    /**
     * 判断当前用户是否拥有该 URL 的权限
     * @param req
     * @param auth
     * @return
     */
    boolean hasPermission(HttpServletRequest req, Authentication auth);
}

```

- UserServiceImpl.java

```java
@Service
public class UserServiceImpl implements UserService {

    /**
     * 判断请求的 URI 是否在用户拥有权限的集合中
     * @param req
     * @param auth
     * @return
     */
    @Override
    public boolean hasPermission(HttpServletRequest req, Authentication auth) {
        Collection<? extends GrantedAuthority> objs  = auth.getAuthorities();
        String requestURI = req.getRequestURI();
        return objs.contains(new SimpleGrantedAuthority(requestURI));
    }
}

```

- 修改配置类 

```java
//userServiceImpl、request、authentication 是 Spring IOC 存在的对象
.anyRequest().
    access("@userServiceImpl.hasPermission(request,authentication)");
// .anyRequest().authenticated();           // 所有请求都需被认证,登录
```

- 修改 UserDetailsServiceImpl.java

```java
 // ROLE_ 开头表示权限
// 将数据库中查询的权限放入下面 List 即可
return new User(
	username,
	password, AuthorityUtils.
	commaSeparatedStringToAuthorityList("admin,normal,ROLE_abc,/main.html")
);
```

## 基于注解的访问控制 ##

> Spring Security 提供了一些访问控制的注解。这些注解默认是都不可用的，需要通过 @EnableGlobalMethodSecurity 进行开启后使用。

如果设置的条件允许，程序正常执行。如果不允许，会报 500 错误

这些注解可以写到 Service 接口或者方法，也可以写到 Controller 或 Controller 的方法上。通常情况下写在控制器方法上，控制器接口 URL是否允许被访问

### @Secured 注解 ###

> 专门用于判断是否具有角色的，能写在方法或类上。参数要以 `ROLE_`开头。
- LoginController.java

```java
/**
     * 跳转到登录成功界面
     * @return
     */
@Secured("ROLE_abc")
@PostMapping("/toMain")
public String toMain(){
    return "redirect:/main.html";
}

```


- Spring boot 启动类

```java
@SpringBootApplication
@EnableGlobalMethodSecurity(securedEnabled = true)
public class App 
{
    public static void main( String[] args ) {
        SpringApplication.run(App.class,args);
    }
}
```

- SecurityConfig.java

```java
/**
 * Spring Security 配置
 */
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    // 重写 configure 方法
    // 注意 configure 方法有两个，选择参数 HttpSecurity
    @Override
    protected void configure(HttpSecurity http) throws Exception {

        //表单控制
        http.formLogin()
                .loginProcessingUrl("/login")
                .usernameParameter("name")        //自定义表单参数用户名
                .passwordParameter("pwd")         //自定义表单参数密码
                .loginPage("/login.html")         // 登录界面
                .successForwardUrl("/toMain");     //跳转到主界面

        // URL 认证
        http.authorizeRequests()
                .antMatchers("/login.html").permitAll()  // 放行 login.html
                .anyRequest().authenticated();           // 所有请求都需被认证,登录

        //关闭 csrf 防护
        http.csrf().disable();

    }

    /**
     * 将创建的 BCryptPasswordEncoder 对象放入 Spring IOC 容器中
     *
     * @return
     */
    @Bean
    public PasswordEncoder getPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }
}

```

### @PreAuthorize 和 @PostAuthorize ###

> 都是作用在方法或者类级别注解

|                |                                                              |
| -------------- | ------------------------------------------------------------ |
| @PreAuthorize  | 表示访问方法或类在执行之前先判断权限，太多情况下都是使用这个注解，注解的参数和 access () 方法参数取值相同，都是权限表达式 |
| @PostAuthorize | 表示访问方法或类在执行结束后判断权限，用的很少               |

- Spring Boot 启动类：开启注解

```
@SpringBootApplication
@EnableGlobalMethodSecurity(securedEnabled = true,prePostEnabled = true)
public class App 
{
    public static void main( String[] args ) {
        SpringApplication.run(App.class,args);
    }
}
```

- LoginController .java

```java
//@Secured("ROLE_abc")
@PreAuthorize("hasRole('ROLE_abc')") // hasrole() 方法不可以以 ROLE 开头，但是 @PreAuthorize 可以以 ROLE_ 开头,依旧严格区分大小写
@PostMapping("/toMain")
public String toMain(){
	return "redirect:/main.html";
}
```

## Remoember Me 功能实现 ##

> 在登录时添加 remeber-me 复选框，取值为 true
>
> Spring Security 会自动把用户信息存储到数据源中，以后就可以不登录直接进行访问，默认有效时间为2周

- 修改 pom 文件

```xml

```

- 修改配置类

```java

```

##  thymeleaf 在 Spring Security 中使用  ##

1. **在 HTML 页面显示认证**

- 修改 pom 文件

```xml
 <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <!--  thymeleaf 在 Spring Security 中使用 -->
        <dependency>
            <groupId>org.thymeleaf.extras</groupId>
            <artifactId>thymeleaf-extras-springsecurity5</artifactId>
        </dependency>
```

- 修改 main.html

> 添加 thymeleaf 头部约束

```html
<!DOCTYPE html>
<html lang="en"
        xmlns="http://www.w3.org/1999/xhtml"
        xmlns:th="http://www.thymeleaf.org"
        xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecuriy5">
<head>
    <meta charset="UTF-8">
    <title>主界面</title>
</head>
<body>
        欢迎 <span th:text="${session.name}"></span>登录成功!!!
        <br/>
        <a th:href="@{/main1.html}">点击跳转到 main1 </a><br/>

        <br/>

        登录账号 <span sec:authentication="name"></span><br/>
        主体中获取登录账号：<span sec:authentication="principal.username"></span><br/>
        获取凭证:<span sec:authentication="credentials"></span><br/>
        权限和角色:<span sec:authentication="authorities"></span><br/>
        客户端IP:<span sec:authentication="details.remoteAddress" ></span>
</body>
</html>
```

2. **权限判断**

> 在 html 页面中可以使用 `sec:authorize="表达式"` 进行权限控制，判断是否显示某些内容。表达式的内容和 access（表达式）的用法相同

- 当前登录用户的角色和权限

```java
 return new User(
 	username,
 	password, 
 	AuthorityUtils.
 	commaSeparatedStringToAuthorityList("admin,normal,ROLE_abc,/select,/delete")
 	);
```

- html 页面，不同权限用户显示不同的效果

```html
 根据权限判断：<br/>
<button sec:authorize="hasAuthority('/select')">查找</button>
<button sec:authorize="hasAuthority('/insert')">插入</button>
<button sec:authorize="hasAuthority('/update')">更新</button>
<button sec:authorize="hasAuthority('/delete')">删除</button><br/>

根据角色判断：<br/>
<button sec:authorize="hasRole('abc')">新增</button>
<button sec:authorize="hasRole('abc')">删除</button>
<button sec:authorize="hasRole('abc')">更新</button>
<button sec:authorize="hasRole('abc')">查找</button>
```

## 退出登录 ##

> spring security 提供了 `/logout` 退出

```html
    <a th:href="@{/logout}">退出</a>
```

- 修改 Spring Security 配置类

```java
//退出登录
http.logout()
//  .logoutUrl("/user/logout")  //指定退出的 URL 地址，不推荐
.logoutSuccessUrl("/login.html");  //退出跳转到指定的 HTML 页面
```

## CSRF ##

> CSRF（Cross-site request forgery） 跨站请求伪造，也称为 “one Click  Attack”或者 “Session Riding”。**通过伪造用户请求访问受信任站点的非法请求访问**
>
> 跨域：只要网络协议，IP 地址，端口中任何一个不相同就是跨域请求。
>
> 客户端与服务器进行交互时，由于 HTTP 协议本身无状态协议，所以引入了 cookies 进行记录客户端身份。在 cookies 中会存放 session id 用来识别客户端身份。在跨域的情况下，session id 可能被第三方恶意劫持，通过 这个 session id 向服务器发起请求时，服务端会认为这个请求是合法的，可能发生很多意想不到的事情。

`http.csrf.disable` ；如果没有这行代码导致用户用户无法被认证。表示关闭 csfr 防护

```html
<input type="hidden"  name="_csrf" 
               th:value="${_csrf.token}" th:if="${_csrf}" />
```

注意： 新版本，不会手动添加隐藏域提交 _csrf.token 



注意：从Spring Security 4.0开始，默认情况下会启用CSRF保护，以防止CSRF攻击应用程序**，Spring Security CSRF会针对PATCH，POST，PUT和DELETE方法进行防护**。**所以在默认配置下，即便已经登录了，页面中发起PATCH，POST，PUT和DELETE请求依然会被拒绝，并返回403，需要在请求接口的时候加入csrfToken才行。**
如果你使用了freemarker之类的模板引擎或者jsp，针对表单提交，可以在表单中增加如下隐藏域

[spring security CSRF防护](https://blog.csdn.net/yjclsx/article/details/80349906)

下面是我之前遇到的坑，post 方式获取树形菜单访问不了 403，但是在浏览器地址栏上 get 方式访问可以获取到菜单中的数据。

```js
/*发送 ajax 请求，获取树形菜单的数据*/
$('#show_Menu').tree({   
    url:'/menu/showMenu', 
    //默认发送 POST 请求,会被 Spring Security 拦截，原因是没有携带 csrf Token    
    method:'get'}
);
```

解决方案：将 Post 方式请求改为 get 方式请求

---

其他解决方案

1. ajax方式请求提交

   **step1**：设置请求头

```html
<meta  name = "_csrf" th:content = "${_csrf.token}" />
 <!-- 默认标题名称是X-CSRF-TOKEN  -->
    <meta  name = "_csrf_header"  th:content = "${_csrf.headerName}" />
```

​		**step2**：通过 jquey 方式获取头部的值

```js
 var token = $("meta[name='_csrf']").attr("content");
  var header = $("meta[name='_csrf_header']").attr("content");
```

​    	**step3:**将名为 _csrf 的值 token放入请求参数

```js
var token = $("meta[name='_csrf']").attr("content");
var header = $("meta[name='_csrf_header']").attr("content");
$.ajax({
	url:url,
	type:'POST',
	async:false,
	dataType:'json',    //返回的数据格式：json/xml/html/script/jsonp/text
	beforeSend: function(xhr) {
		xhr.setRequestHeader(header, token);  //发送请求前将csrfToken设置到请求头中
	},
	success:function(data,textStatus,jqXHR){
	}
});

/*************/
$('#role_table').datagrid({
    url: '/role/showRoles',
    queryParams: {
  	  _csrf:token
    },
    ...
    }
}
```

2. form 表单提交

```html
<input type="hidden"  name="_csrf" 
               th:value="${_csrf.token}" th:if="${_csrf}" />
```

