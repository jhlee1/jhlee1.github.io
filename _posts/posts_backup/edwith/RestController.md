---
title: "RestController"
date: 2019-05-06T23:23:40+09:00
archives: "2019"
tags: ["spring", "edwith", "webprogramming"]
author: Joohan Lee
---

## RestController

### 1. @RestController

- Spring 4에서 RestAPI 또는 Web API를 개발하기 위해 등장
- 이전 버전의 @Controller와 @ResponseBody를 포함
- Rest API를 구현하기 위해 MessageConverter가 중요함



### 2. MessageConvertor

- 자바 객체와 HTTP 요청 / 응답 바디를 변환하는 역할 (주로 Json)
- @ResponseBody, @RequestBody
- @EnableWebMvc 로 인한 기본 설정
- WebMvcConfigurationSupport 를 사용하여 Spring MVC 구현
- Default MessageConvertor 를 제공



### 3. JSON 응답하기

- Controller의 메소드에서는 JSON으로 변환될 객체를 반환
- @EnableWebMvc에서 jackson library를 기본 설정으로 사용함
- jackson라이브러리를 추가하지 않으면 JSON메시지로 변환할 수 없어 500 Error 발생
- 사용자가 임의의 메시지 컨버터(MessageConverter)를 사용하도록 하려면 WebMvcConfigurerAdapter의 configureMessageConverters메소드를 @Override해야함



ex) **GuestbookAPIController.java**

```java
package lee.joohan.guestbook.controller;

import lee.joohan.guestbook.dto.Guestbook;
import lee.joohan.guestbook.service.GuestbookService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import javax.servlet.http.HttpServletRequest;
import java.util.*;

@RestController
@RequestMapping(path="/guestbooks")
public class GuestbookAPIController {
    @Autowired
    GuestbookService guestbookService;

    @GetMapping
    public Map<String, Object> getList(@RequestParam(name="start", required = false, defaultValue = "0") int start) {
        List<Guestbook> list = guestbookService.getGuestbooks(start);
        int count = guestbookService.getCount();
        int pageCount = count / GuestbookService.LIMIT;

        if (count % GuestbookService.LIMIT > 0) {
            pageCount++;
        }

        List<Integer> pageStartList = new ArrayList<>();

        for (int i = 0;i < pageCount; i++) {
            pageStartList.add(i * GuestbookService.LIMIT);
        }

        Map<String, Object> map = new HashMap<>();
        map.put("list", list);
        map.put("count", pageCount);
        map.put("pageStartList", pageStartList);

        return map;
    }

    @PostMapping
    public Guestbook write(@RequestBody Guestbook guestbook, HttpServletRequest request) {
        String clientIp = request.getRemoteAddr();
        Guestbook resultGuestbook = guestbookService.addGuestbook(guestbook, clientIp);

        return resultGuestbook;
    }

    @DeleteMapping("/{id}")
    public Map<String, String> delete(@PathVariable(name = "id") Long id, HttpServletRequest request) {
        String clientIp = request.getRemoteAddr();
        int deleteCount = guestbookService.deleteGuestbook(id, clientIp);
        
        return Collections.singletonMap("success", deleteCount > 0 ? "true" : "false");
    }
}
```



### 4. 테스트 하기

- Chrome extentions에 Restlet Client 설치하고 실행
- Header의 Content-type을 application/json으로 설정



원본 강의: https://www.edwith.org/boostcourse-web