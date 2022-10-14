## Spring MVC DTO에 Setter가 필요한 경우

### 항상 헷갈리는

@ModelAttribute로 매핑되는 DTO는 Setter가 필요하고, @RequestBody로 매핑되는 DTO는 Setter가 불필요하다.

### Mapper 객체가 달라서

RequestBody : `Jackson2HttpMessageConverter`     
ModelAttribute : `WebDataBinder`
