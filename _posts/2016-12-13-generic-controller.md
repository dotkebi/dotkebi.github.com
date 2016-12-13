---
layout: post
title:  "Generic contoller set"
date:   2016-12-13 15
categories: java spring generic
---

## Intro

`generic controller`가 필요한가? 라고 물어본다면 글쎄? 정도로 흘려버립니다. 잘 쓰면 유용하지 않을까요?
어느 날 `controller, service, dao, model`들을 만들고 있는데 이 모델 클래스 도메인 별로 겹치는게 많은데 굳이 따로 만들 필요가 있나? 하는 생각이 들었고 조금씩 만들어 보게 되었습니다.

`generic`에 대한 설명은 레퍼런스를 참고하도록 합시다.

## 1. sources

### 1.1 controller
``` java
package io.github.dotkebiweb.controller;

import io.github.dotkebiservice.hold.AbstractService;
import io.github.dotkebiservice.hold.TransformFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.data.web.PageableDefault;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.bind.support.SessionStatus;

import javax.validation.Valid;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;

/**
 * Generic Controller
 * @author by dotkebi@gmai.com
 */
public abstract class AbstractController<T, K extends Serializable> {

    protected AbstractService<T> service;

    @Autowired
    public AbstractController(AbstractService<T> service) {
        this.service = service;
    }

    @RequestMapping(value = "", method = RequestMethod.GET)
    public String fetchList(
            @PageableDefault(sort = { "id" }, direction = Sort.Direction.DESC) Pageable page
            , Model model
    ) {
        Page<T> contents = service.fetchAll(page);
        model.addAttribute("dataList", contents.getContent());
        return getPageHeader() + "List";
    }

    @RequestMapping(value = "/{id}", method = RequestMethod.GET)
    public String fetch(
            @PathVariable K id
            , Model model
    ) {
        model.addAttribute("data", service.fetch(id));
        return getPageHeader() + "Detail";
    }

    @RequestMapping(value = "/modify/{id}", method = RequestMethod.GET)
    public String modify(
            @PathVariable K id
            , Model model
    ) throws ClassNotFoundException, IllegalAccessException, InstantiationException {
        model.addAttribute("data", service.fetch(id));
        model.addAttribute("title", getTitle() + " 수정");
        return getPageHeader() + "Modify";
    }

    @RequestMapping(value = "/add", method = RequestMethod.GET)
    public String add(
            Model model
    ) throws ClassNotFoundException, IllegalAccessException, InstantiationException {
        Type type = ((ParameterizedType) getClass().getGenericSuperclass()).getActualTypeArguments()[0];
        model.addAttribute("data", ((Class) type).newInstance());
        model.addAttribute("title", getTitle() + " 추가");
        return getPageHeader() + "New";
    }

    @RequestMapping(value = "", method = RequestMethod.POST)
    public String saveOrUpdate(
            @ModelAttribute @Valid T t
            , BindingResult bindingResult
            , SessionStatus sessionStatus
            , Model model
    ) {
        if (bindingResult.hasErrors()) {
            model.addAttribute("errors", bindingResult.getAllErrors());
            String modifier = "/add";
            if (!t.getId() > 0) {
                modifier = "/modify/" + t.getId();
            }
            return "redirect:" + getAPIUrl() + modifier;
        }
        if (t.getId() > 0) {
            service.save(t);
        }
        else {
            service.update(t);
        }
        sessionStatus.setComplete();
        return "redirect:" + getAPIUrl();
    }

    @RequestMapping(value = "/{id}", method = RequestMethod.DELETE)
    public @ResponseBody Boolean delete(
            @PathVariable(value = "id") K id
    ) {
        service.delete(id);
        return true;
    }

    protected abstract String getPageHeader();
    protected abstract String getAPIUrl();
    protected abstract String getTitle();

}

```

### 1.2. service

``` java

package io.github.dotkebi.service;

import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;

import java.io.Serializable;

/**
 * generic service
 * @author by dotkebi@gmai.com
 */
public interface AbstractService<T, K extends Serializable> {

    Page<T> fetchAll(Pageable pageRequest);

    T fetch(K id);

    void save(T t);

    void update(T t);

    void delete(K id);

}
```

### 1.3. service implementation

``` java
package io.github.dotkebi.service;

import io.github.dotkebi.service.AbstractService;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;

import java.io.Serializable;

/**
 * generic service implementation
 * @author by dotkebi@gmail.com
 */
public abstract class AbstractServiceImpl<T, K extends Serializable> implements AbstractService<T, K> {

    protected JpaRepository<T, K> dao;

    public AbstractServiceImpl(JpaRepository<T, K> dao) {
        this.dao = dao;
    }

    @Override
    public Page<T> fetchAll(Pageable pageRequest) {
        return dao.findAll(pageRequest);
    }

    @Override
    public T fetch(K id) {
        return dao.findOne(id);
    }

    @Override
    public void save(T t) {
        dao.save(t);
    }

    @Override
    public void update(T t) {
        dao.save(t);
    }

    @Override
    public void delete(K id) {
        dao.delete(id);
    }

}

```

### 1.4. dao

``` java
package io.github.dotkebi.dao;

import io.github.dotkebi.model.Test;
import org.springframework.data.jpa.repository.JpaRepository;

/**
 * @author by dotkebi@gmail.com
 */
public interface TestDao extends JpaRepository<Test, Long> {
}
```

## 2. 예제

``` java
package io.github.dotkebi.domain;

import lombok.*;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

/**
 * @author by dotkebi@gmail.com
 */
@Data
@NoArgsConstructor
@AllArgsConstructor
@Entity
public class Test {

    @Id
    @GeneratedValue
    private Long id;

    @Column
    private String name;

    @Column
    private String addr;

}
```

``` java
package io.github.dotkebi.controller;

import io.github.dotkebi.domain.Test;
import io.github.dotkebi.service.TestServiceImpl;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;

import javax.annotation.PostConstruct;
import java.util.List;

/**
 * @author by dotkebi@gmail.com
 */
@Controller
@RequestMapping(value = "")
public class TestController extends AbstractController<Test, Long> {

    @Autowired
    public TestController(TestServiceImpl service) {
        super(service);
    }

    @Override
    protected String getPageHeader() {
        return "test";
    }

    @Override
    protected String getAPIUrl() {
        return "/test";
    }

    @Override
    protected String getTitle() {
        return "test";
    }
}
```

``` java
package io.github.dotkebi.service;

import io.github.dotkebi.domain.Test;
import io.github.dotkebi.dao.TestDao;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Service;

/**
 * @author by dotkebi@gmail.com
 */
@Service
public class TestServiceImpl extends AbstractServiceImpl<Test, Long> {

    @Autowired
    public TestServiceImpl(TestDao dao) {
        super(dao);
    }
    
}
```
