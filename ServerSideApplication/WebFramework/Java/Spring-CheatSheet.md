# Spring CheatSheet | Spring 概念、使用与实践

传统的 Spring MVC 基于 Servlet API 构建，使用单请求单线程处理的同步阻塞型模型；而 Spring WebFlux 则是 Reactive Stack，能够充分利用现代多核处理器的特性，从底层机制上保证了对于海量并发请求处理的能力。WebFlux 使用 Netty, Servlet 3.1+ Containers 替代传统的 Servlet Containers，使用 Reactive Stream Adapters 替代 Servlet API，使用 Spring Security Reactive 替代 Spring Security，使用 Spring Data Reactive Repositories 替代 Spring Data Repositories.
