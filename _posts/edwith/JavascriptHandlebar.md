---
title: "Javascript Handlebar"
date: 2019-05-09T00:01:16+09:00
archives: "2019"
tags: ["edwith", "javascript", "webProgramming"]
author: Joohan Lee
---

## Handlebar

### 1. handlebar를 이용해서 template 작성하기 

- 기본 예제

```html
<html>
	<head>
    	<style>
            li { list-style : none;}
        </style>
    </head>
	<body>
        <h1>Template using handlebar</h1>

        <section class="show">

        </section>
        <script type="myTemplate" id="listTemplate">
        	<li>
        		<div>Author: {{name}}</div>
        		<div class="content">{{content}}</div>
        		<div>likes: <span> {{like}} </span></div>
        		<div class="comment">
        			<div>{{comment}}</div>
        		</div>
        	</li>
        </script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/handlebars.js/4.1.2/handlebars.min.js"></script>
        <script>
            var data = {
                "id": 88,
                "name": "John",
                "content": "new post!",
                "like": 5,
                "comment": "This is a comment"
            };
        	var template = document.querySelector("#listTemplate").innerText;

            //pre-compile
        	var bindTemplate = Handlebars.compile(template); //bindTemplate은 메서드입니다.
            var resultHTML = bindTemplate(data);
            console.log(resultHTML);
            var show = document.querySelector(".show")
            show.innerHTML = resultHTML;
        </script>
    </body>
</html>

```

### 2. 배열이 포함된 데이터 처리

- comment가 여러개인 경우 each를 이용해서 처리

```html
<html>
	<head>
    	<style>
            li { list-style : none;}
        </style>
    </head>
	<body>
        <h1>Template using handlebar</h1>

        <section class="show">

        </section>
        <script type="myTemplate" id="listTemplate">
        	<li>
        		<div>Author: {{name}}</div>
        		<div class="content">{{content}}</div>
        		<div>likes: <span> {{like}} </span></div>
        		<div class="comment">
        			<h3>List of comments</h3>
        			{{#each comment}}
        			<div>{{@index}}th comment: {{this}}</div>
        			{{/each}}
        		</div>
        	</li>
        </script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/handlebars.js/4.1.2/handlebars.min.js"></script>
        <script>
            var data = {
                "id": 88,
                "name": "John",
                "content": "new post!",
                "like": 5,
                "comment": ["This is a comment", "It is also a comment", "comment3"]
            };
        	var template = document.querySelector("#listTemplate").innerText;

            //pre-compile
        	var bindTemplate = Handlebars.compile(template); //bindTemplate은 메서드입니다.
            var resultHTML = bindTemplate(data);
            console.log(resultHTML);
            var show = document.querySelector(".show")
            show.innerHTML = resultHTML;
        </script>
    </body>
</html>

```

### Data가 여러개인 경우

- foreach 혹은 reduce를 이용해서 합치기

```html
<html>
	<head>
    	<style>
            li {
							list-style : none;
							padding: 3%;
							border-top: 1px solid grey;
						}
        </style>
    </head>
	<body>
        <h1>Template using handlebar</h1>

        <section class="show">

        </section>
        <script type="myTemplate" id="listTemplate">
        	<li>
        		<div>Author: {{name}}</div>
        		<div class="content">{{content}}</div>
        		<div>likes: <span> {{like}} </span></div>
        		<div class="comment">
        			<h3>List of comments</h3>
        			{{#each comment}}
        			<div>{{@index}}th comment: {{this}}</div>
        			{{/each}}
        		</div>
        	</li>
        </script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/handlebars.js/4.1.2/handlebars.min.js"></script>
        <script>
            var data = [
							{"id": 88, "name": "John", "content": "new post!", "like": 5, "comment": ["This is a comment", "It is also a comment", "comment3"]},
							{"id": 77, "name": "Smith", "content": "new post!!", "like": 4, "comment": ["This is a comment", "It is also a comment", "comment3"]},
							{"id": 66, "name": "Joseph", "content": "new post###", "like": 3, "comment": ["This is a comment", "It is also a comment", "comment3"]},
							{"id": 55, "name": "Micheal", "content": "new post$$$$", "like": 2, "comment": ["This is a comment", "It is also a comment", "comment3"]}
					];
        	var template = document.querySelector("#listTemplate").innerText;

            //pre-compile
        	var bindTemplate = Handlebars.compile(template); //bindTemplate은 메서드입니다.

					var resultHTML = data.reduce(function(prev, next) {
						return prev + bindTemplate(next);
					}, "")

            var show = document.querySelector(".show")
            show.innerHTML = resultHTML;
        </script>
    </body>
</html>
```

### 4.  조건문

- {{#if (condition) }} {{else}} {{/if}} 사용하기

```html
<html>
	<head>
    	<style>
            li {
							list-style : none;
							padding: 3%;
							border-top: 1px solid grey;
						}
        </style>
    </head>
	<body>
        <h1>Template using handlebar</h1>

        <section class="show">

        </section>
        <script type="myTemplate" id="listTemplate">
        	<li>
        		<div>Author: {{name}}</div>
        		<div class="content">{{content}}</div>
        		<div>likes: <span> {{like}} </span></div>
        		<div class="comment">
        			<h3>List of comments</h3>
							{{#if comment}}
        				{{#each comment}}
        					<div>{{@index}}th comment: {{this}}</div>
        				{{/each}}
							{{else}}
								<div> There is no comment yet</div>
							{{/if}}
        		</div>
        	</li>
        </script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/handlebars.js/4.1.2/handlebars.min.js"></script>
        <script>
            var data = [
							{"id": 88, "name": "John", "content": "new post!", "like": 5, "comment": ["This is a comment", "It is also a comment", "comment3"]},
							{"id": 77, "name": "Smith", "content": "new post!!", "like": 4, "comment": ["This is a comment", "It is also a comment", "comment3"]},
							{"id": 66, "name": "Joseph", "content": "new post###", "like": 3, "comment": ["This is a comment", "It is also a comment", "comment3"]},
							{"id": 55, "name": "Micheal", "content": "new post$$$$", "like": 2, "comment": []}
					];
        	var template = document.querySelector("#listTemplate").innerText;

            //pre-compile
        	var bindTemplate = Handlebars.compile(template); //bindTemplate은 메서드입니다.

					var resultHTML = data.reduce(function(prev, next) {
						return prev + bindTemplate(next);
					}, "")

            var show = document.querySelector(".show")
            show.innerHTML = resultHTML;
        </script>
    </body>
</html>

```



### 5. Helper function

- Handlebar에게 function을 정의해서 추가할 수 있음
- 좋아요 갯수가 5개 이상이면 "Contratulations!" 추가하기
- Handlebar에 Helper function 등록

```javascript
Handlebars.registerHelper("likes", function(like) {
	if(like > 4) {
		return "<span>Congratulations! you have " + like  + "likes</span>";
	} else if (like < 1) {
        return "No one likes your post yet.";
    } else {
        return "There are " + like + "likes";
    }
});
```

- 예제

```html
	<html>
		<head>
	    	<style>
	            li {
								list-style : none;
								padding: 3%;
								border-top: 1px solid grey;
							}
	        </style>
	    </head>
		<body>
	        <h1>Template using handlebar</h1>
	        <section class="show">
	        </section>
	        <script type="myTemplate" id="listTemplate">
	        	<li>
	        		<div>Author: {{name}}</div>
	        		<div class="content">{{content}}</div>
	        		{{#likes like}}
								{{like}}
							{{/likes}}
	        		<div class="comment">
	        			<h3>List of comments</h3>
								{{#if comment}}
	        				{{#each comment}}
	        					<div>{{@index}}th comment: {{this}}</div>
	        				{{/each}}
								{{else}}
									<div> There is no comment yet</div>
								{{/if}}
	        		</div>
	        	</li>
	        </script>
	        <script src="https://cdnjs.cloudflare.com/ajax/libs/handlebars.js/4.1.2/handlebars.min.js"></script>
	        <script>
	            var data = [
								{"id": 88, "name": "John", "content": "new post!", "like": 5, "comment": ["This is a comment", "It is also a comment", "comment3"]},
								{"id": 77, "name": "Smith", "content": "new post!!", "like": 4, "comment": ["This is a comment", "It is also a comment", "comment3"]},
								{"id": 66, "name": "Joseph", "content": "new post###", "like": 3, "comment": ["This is a comment", "It is also a comment", "comment3"]},
								{"id": 55, "name": "Micheal", "content": "new post$$$$", "like": 2, "comment": []}
						];

						Handlebars.registerHelper("likes", function(like) {
							if(like > 4) {
								return "<span>Congratulations! you have " + like  + " likes</span>";
							} else if (like < 1) {
						  	return "No one likes your post yet.";
					    } else {
					      return "There are " + like + " likes";
						  }
						});

	        	var template = document.querySelector("#listTemplate").innerText;

	            //pre-compile
	        	var bindTemplate = Handlebars.compile(template); //bindTemplate은 메서드입니다.

						var resultHTML = data.reduce(function(prev, next) {
							return prev + bindTemplate(next);
						}, "")

	            var show = document.querySelector(".show")
	            show.innerHTML = resultHTML;
	        </script>
	    </body>
	</html>

```